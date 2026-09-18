- **Source**: Excerpt, §§2.1–2.5 on looped transformers (Figures 5–11). *(No byline in the text provided; the author refers to "my LLM Architecture Gallery" and coverage on Substack Notes, which points to Sebastian Raschka.)*
- **One-liner**: A looped transformer runs the hidden states through the **same** blocks more than once, so effective depth rises while the weight count does not — but it saves **only** weight memory: forward compute, backprop, and KV cache all scale with **block applications**, not with distinct blocks. The interesting designs are therefore the ones that add **adaptivity**, deciding per token how many passes to spend.
- ## 1. Vocabulary (worth pinning down first)
	- **Transformer block** — a unit containing attention, a feedforward module, normalization, and shortcut connections. *Papers usually call these "transformer layers."*
	- **Stack** — a sequence of transformer blocks.
	- **Block application** — running an input through a transformer block **once**.
	- The whole subject is the gap between these last two: **a block is a set of weights; a block application is a unit of work.** Looping decouples them, and every cost question below is really "which of the two does this resource scale with?"
- ## 2. The idea, and the concrete example
	- **Looped transformer** = pass the intermediate representations through the **same** transformer blocks multiple times instead of once. Versus simply adding more blocks, the trick is that **the weights stay the same across passes**.
	- Not new — the basic idea is already in the **Universal Transformers** paper (2018).
	- **Nanbeige4.2-3B** (open-weight, released July) is the simplest case: an ordinary-looking transformer with one extra arrow looping from the top of the stack back to the bottom.
		- Text is tokenized → embeddings → **22 transformer blocks, each with its own weights**.
		- After the first pass, the hidden states are **fed back through the same 22 blocks**: block 1 again, then block 2, … up to block 22.
		- Unrolled, that is **44 block applications** — application 23 uses block 1's weights, application 24 uses block 2's, and so on.
		- Net effect: **effective depth 22 → 44 without a second set of transformer weights.**
	- **Why two passes and not three?** The paper is thin on detail but says two was the most efficient setup: **2 → 3 can improve modeling performance, but the extra compute was not worth it.** More passes also slowed training and made optimization less stable.
	- **Trained from scratch beat upcycling** — converting an already-pretrained conventional transformer into a looped one worked less well than training the looped architecture directly.
- ## 3. The accounting — what looping actually buys
	- **The unrolled picture** (Figures 5–7, redrawn):
	  ```
	    CONVENTIONAL 44 blocks              LOOPED 22 blocks x 2
	    ----------------------              --------------------
	      block 44   weights W44              app 44    weights W22
	         :                                   :
	      block 23   weights W23              app 23    weights W1    <-- reuse
	      block 22   weights W22              app 22    weights W22
	         :                                   :
	      block  1   weights W1               app  1    weights W1
	  
	      44 sets of weights                  22 sets of weights   <-- THE saving
	      44 block applications               44 block applications
	      44 KV caches                        44 KV caches
	  ```
	- | Resource | Scales with | Saved by looping? |
	  |---|---|---|
	  | **Transformer-block parameters** | distinct blocks | **Yes — roughly half.** The only real win: less memory to store weights |
	  | **Forward compute** | block applications | **No.** Still 44 applications per forward pass |
	  | **Backward compute** | block applications | **No.** Gradients flow through **both** repetitions of the shared stack |
	  | **Optimizer state** | distinct blocks | **Yes** — fewer distinct parameters to update |
	  | **KV cache** | block applications | **No** — see below |
	- **Why the KV cache is not shared.** Even with shared weights, the **intermediate states entering a block differ on the second pass**, so the resulting keys and values differ too. Applications 1 and 23 use the same weights but need **their own cache entries**. A 22-block stack run twice therefore has **the same KV cache requirement as a conventional 44-block model**.
		- Nanbeige **tried** sharing the KV cache between passes. It halved the cache, and **the model performed worse** — they shipped the separate-cache version. (For why KV cache capacity is the binding constraint in serving, see [[2026-09-pmpp-ch1-cpu-vs-gpu-design-philosophies]] §15–17.)
	- **The one asterisk on "parameters":** embedding and output layers are usually large and sit **outside** this comparison. For Nanbeige4.2-3B they are **~25% of the 3B total**; tying them would cut that to **~12.5%**. Looping shrinks the *middle* of the model, not the ends.
	- **The blunt summary**: looping is a **parameter-efficiency** technique, not a compute-efficiency one. You pay 44 blocks' worth of compute and cache to store 22 blocks' worth of weights. Whether that is a good trade depends entirely on whether weight memory was your binding constraint.
- ## 4. Making the loop count flexible
	- The above is the *fixed*-loop case. Everything interesting after it is about **spending a different number of passes on different tokens** — which is also the only route to looping actually **saving** compute rather than merely relocating it.
	- **Universal Transformer (2018)** — repeatedly applies the **same single block** (rather than a stack, as Nanbeige does). Steps can be fixed, but the paper explores **adaptive halting**:
		- A small **trained function outputs a halting probability** for each position at each step.
		- Probabilities are **summed over successive loops**; a position **stops looping once the sum exceeds a threshold**.
		- A **maximum loop count** caps the computation as a safety net.
		- The point: **allocate compute to the tokens that benefit from it.**
	- **ByteDance Ouro** — a more extreme fixed case with an adaptive exit. **Ouro-Thinking 2.6B applies the same 48-block stack four times: 192 block applications from 48 distinct blocks.** A **learned exit gate** assigns probabilities to the different exits, and a threshold on cumulative probability decides **which pass supplies the output** — adaptive halting, which Nanbeige did not use.
		- **Practical caveat**: the released Hugging Face implementation **computes all configured passes before selecting an output**, so in practice the loop count is effectively hard-coded to 4. *(Adaptive in principle, not yet adaptive in the shipped code.)*
	- **Mixture-of-Recursions (MoR, 2025)** — a more sophisticated Universal Transformer. The repeated stack is called a **recursion block** and sits **between separate first and last blocks** (Layer 0 and Layer L−1); the innovation is **how the per-token loop count is decided**.
		- Instead of a halting probability accumulated step by step, MoR uses **a small learned router** — the same idea as mixture-of-experts routing, except the decision is **how many times to apply the shared stack** rather than which expert to use.
		- **Crucially, the router reads the token's hidden representation**, which carries context. So this is **not** a property of the token type: the word "People" does not always get three passes; the count depends on where it appears and what came before it.
		- Two routing schemes:
		- | | **Expert-choice routing** | **Token-choice routing** |
		  |---|---|---|
		  | **Who decides** | Each recursion step selects which tokens it will process | One router decides at the very beginning |
		  | **How** | Tokens that exit are excluded from later steps | Each token is assigned a path of 1, 2, or 3 passes up front |
		  | **Flavor** | Iterative, step-by-step filtering | A single routing decision per token |
		- Weights are reused across passes as before; the added flexibility is purely in **how much computation each token receives**. **Model and routers are trained together**, so the model learns to work with the different paths.
- ## 5. Does it work?
	- The MoR paper compares **Vanilla** (regular transformer), **Recursive** (fixed recursion), and **MoR** on validation loss across **four model scales × three training-compute budgets**.
	- **At the smallest scale, the regular transformer wins.** At larger scales MoR catches up and often does better — **especially at the smaller training budgets**. At the largest budget several curves nearly coincide.
	- A subtlety that explains part of the gain: **equal training compute does not mean an equal number of training tokens.** By skipping computation for some tokens, MoR **processes more tokens within the same budget**.
	- **Verdict as stated**: looped transformers can improve model quality **at a fixed compute budget, if the model is large enough**.
	- **The methodological lesson the author draws, which is arguably the most transferable part**: this only shows up at scale. *Looking at the 135M-parameter model alone, you would have concluded the opposite.*
- ## 6. Takeaways
	- **Separate "how many weights" from "how much work."** A block is weights; a block application is work. Looping halves the former and leaves the latter alone — every claim about looped transformers becomes clear once you ask which quantity is being counted.
	- **Weight sharing does not imply activation sharing.** The KV cache is the crisp demonstration: same weights, different inputs, therefore different keys and values, therefore no saving. Nanbeige's failed experiment in forcing the sharing is the empirical confirmation.
	- **Fixed looping is a memory trade; adaptive looping is a compute trade.** Nanbeige buys smaller weights at unchanged compute. Universal Transformer, Ouro, and MoR add a mechanism to *skip* passes, which is what turns depth-sharing into an actual efficiency win.
	- **Adaptive depth is the training-time cousin of adaptive reasoning effort.** Halting probabilities and MoR routers decide *per token, inside the architecture* how much computation to spend — the same economic idea as the effort settings in [[2026-07-controlling-reasoning-effort-in-llms]], which decide it *per request, via the prompt*. Both are inference-time compute allocation; they differ in granularity and in who chooses.
	- **Beware conclusions drawn at small scale.** The MoR results invert between 135M and the larger models. An architecture comparison is only as good as the largest point on its x-axis.
	- Related: [[2026-09-pmpp-ch1-cpu-vs-gpu-design-philosophies]], [[2026-07-controlling-reasoning-effort-in-llms]], [[2026-04-cs336-lecture-01-overview-tokenization]], [[2026-05-inference-engineering]]
