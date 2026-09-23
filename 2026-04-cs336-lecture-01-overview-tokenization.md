file-created-at:: 2026-07-29

- **Source**: Stanford **CS336: Language Models From Scratch**, Spring 2026 (3rd offering), Lecture 01 — course overview + tokenization unit. Instructors: Percy Liang, Tatsunori Hashimoto; CAs: Marcel Rød, Herman Brunborg, Steven Cao. Delivered as an *executable lecture* (`lecture_01.py` — a Python program whose execution renders the lecture). Spring 2025 lectures on YouTube.
- **One-liner**: The course exists because researchers have become **disconnected from the underlying technology** — and its single organizing principle is **efficiency**: `accuracy = efficiency × resources`, so the real question is always "what's the best model I can build given a fixed compute and data budget?"

- ## Why this course exists
	- **The problem — a receding stack**:
		- 2016: researchers implemented and trained their own models.
		- 2018: researchers downloaded models (e.g. BERT) and fine-tuned them.
		- Today: researchers **prompt API models** (GPT/Claude/Gemini).
	- Moving up abstraction levels boosts productivity, **but** these abstractions are **leaky** (unlike programming languages or operating systems), and fundamental research still requires tearing up the stack. **Full understanding is necessary for fundamental research.**
	- **Course philosophy: understanding via building.**
	- **But there's a problem — industrialization**: frontier models are extremely expensive (GPT-4 reportedly ~**$100M** to train in 2023; xAI built a **230K-GPU** cluster for Grok in 2025), and there are **no public details** on how they're built. The GPT-4 technical report explicitly withholds architecture, model size, hardware, training compute, dataset construction, and training method, citing competitive landscape and safety.
	- Frontier models are out of reach; we *can* build small LMs (<1B params) — but those **might not be representative**:
		- **Example 1**: the fraction of FLOPs spent in attention vs. MLP **changes with scale** (from OPT setups: at 760M, MHA is 35% of FLOPs and FFN 44%; at 175B, MHA drops to 17% and FFN rises to 80%).
		- **Example 2**: **emergence** — some behaviors appear only past a scale threshold (Wei+ 2022).
	- ### What actually transfers to frontier models — three types of knowledge
		- **Mechanics**: how things work (what a Transformer is, how model parallelism works) — **transfers**.
		- **Mindset**: squeezing the most out of hardware, taking scaling seriously — **transfers**.
		- **Intuitions**: which data/modeling decisions yield good accuracy — **only partially transfers** (does not necessarily hold across scales).
	- **On intuitions**: some design decisions simply aren't (yet) justifiable and come from experimentation. The canonical example is Shazeer's SwiGLU paper, whose conclusion says: *"We offer no explanation as to why these architectures seem to work; we attribute their success, as all else, to divine benevolence."*
	- ### The bitter lesson, read correctly
		- **Wrong** interpretation: scale is all that matters, algorithms don't matter.
		- **Right** interpretation: **algorithms that scale are what matter.**
		- $$\text{accuracy} = \text{efficiency} \times \text{resources}$$
		- Efficiency matters *more* at larger scales (you can't afford to be wasteful). Hernandez+ 2020 showed **44× algorithmic efficiency** gain on ImageNet between 2012 and 2019.
		- Framing: **what is the best model one can build given a certain compute and data budget? → maximize efficiency.**

- ## The current LM landscape (a compressed history)
	- **Pre-neural (before 2010s)**: Shannon 1950 (LM to measure the entropy of English); n-gram LMs for MT and speech recognition (Brants+ 2007).
	- **Neural ingredients (2010s)**: LSTM (1997), first neural LM (Bengio+ 2003), seq2seq (2014), Adam (2014), attention (Bahdanau+ 2014), **Transformer** (Vaswani+ 2017), mixture of experts (Shazeer+ 2017), model parallelism (2018–2019).
	- **Early foundation models (late 2010s)**: ELMo (pretraining with LSTMs), BERT (pretraining with Transformer), T5 (11B, cast everything as text-to-text).
	- **Embracing scaling**: GPT-2 (1.5B, first signs of zero-shot) → **scaling laws** (Kaplan+ 2020, "hope/predictability for scaling") → GPT-3 (175B, in-context learning) → PaLM (540B, massive but **undertrained**) → **Chinchilla** (70B, compute-optimal scaling laws).
	- **Open models**, in three tiers of openness:
		- *Early GPT-3 replication attempts*: EleutherAI (The Pile, GPT-J), Meta's OPT (175B, "lots of hardware issues"), BLOOM (176B, focused on data sourcing).
		- *Credible open-weight (weights + paper)*: Llama, Mistral, DeepSeek, Qwen, Kimi, GLM, MiniMax, Xiaomi MiMo — **approaching closed models**.
		- *Open-source (weights + paper + code + data)*: AI2's **Olmo**, NVIDIA's **Nemotron**, **Marin** (open development).
		- Openness matters for trust and innovation — and **ideas from open models are what make CS336 teachable**.
	- ### "What is a language model?" keeps changing
		- 2018 (BERT): something you **fine-tune**. 2020 (GPT-3): something you **prompt**. 2022 (ChatGPT): something you **talk to**. 2026 (agents): something that **acts autonomously**.
		- **The fundamentals are the same** (attention, kernels, optimization); **the specs are different** (longer context, inference efficiency matters even more).

- ## Course logistics & structure
	- **Executable lecture**: a program whose execution delivers the lecture content — lets you view and run code (since everything *is* code) and see the lecture's hierarchical structure.
	- 5-unit class, notoriously heavy. A Spring 2024 evaluation quote: *"The entire assignment was approximately the same amount of work as all 5 assignments from CS224n plus the final project. And that's just the first homework assignment."*
	- **Take it if**: you have an obsessive need to understand how things work; you want to build research-engineering muscles. **Don't take it if**: you need research done this quarter; you want the hottest new techniques (multimodality, RAG — take a seminar); you just want good results in your domain (prompt or fine-tune an existing model).
	- **Assignments**: 5 total, no scaffolding code (but unit tests + adapter interfaces to check correctness); implement locally for correctness, run on cluster for benchmarking; leaderboards for some (e.g. minimize perplexity given a training budget). Compute provided by **Modal**.
	- **AI policy** (notable): coding agents *can* solve all the assignments, **but you won't learn anything**. AI is useful for questions and tutoring; students **must use the course-provided `AGENTS.md`**, which asks the AI to be **pedagogically-minded**.
	- ### Syllabus — 5 units, all about efficiency
	  | Unit | Assignment | Contents |
	  | --- | --- | --- |
	  | **Basics** | A1 | tokenization, model architecture, training |
	  | **Systems** | A2 | kernels, parallelism, inference |
	  | **Scaling laws** | A3 | fitting and extrapolating scaling laws |
	  | **Data** | A4 | evaluation, curation, transformation, filtering, deduplication, mixing |
	  | **Alignment** | A5 | RLHF, RL algorithms, RL systems |
	- **Resources = data + hardware** (compute, memory, communication bandwidth). Today we are **compute-constrained**, so design decisions reflect squeezing the most out of given hardware:
		- *Tokenization*: raw bytes would be elegant, but is compute-inefficient with today's architectures.
		- *Model architecture*: many changes are motivated by reducing memory or FLOPs (sharing KV caches, sliding-window attention).
		- *Data filtering*: avoid wasting compute updating on bad/irrelevant data.
		- *Scaling laws*: use **less** compute on **smaller** models to do hyperparameter tuning.
		- "Tomorrow, we will become **data-constrained**..."

- ## Unit previews
	- ### Basics
		- **The high-level principle — balance three things**: **expressivity** (can represent complex dependencies), **stability** (keep parameter and gradient norms in the goldilocks zone), **efficiency** (runs fast on hardware, training *and* inference).
		- Architecture refinements over the original Transformer: activations (ReLU → **SwiGLU**), positional encodings (sinusoidal → **RoPE**), normalization (LayerNorm, **RMSNorm**, QK-norm, **pre-norm vs post-norm**), attention variants (full, sparse/local, **GQA**, **MLA**), linear attention / state-space models (**Mamba**, Gated DeltaNet), MLP as dense vs **mixture of experts**, and shape (hidden dim, depth, #heads, #experts).
		- Training knobs: loss (e.g. multi-token prediction), optimizer (AdamW, **SOAP**, **Muon**), init scale (Xavier, **muP**), LR schedule (cosine, **WSD**), regularization, batch size (**critical batch size**), MoE load balancing (e.g. aux-free).
	- ### Systems
		- **Resource accounting** is the entry point: `total_flops = 6 * N * D` — e.g. training 70B params on 1T tokens = **4.2e23 FLOPs**.
		- Parameters must move from **memory (HBM) → compute (SMs)**. A **B200** does 2.25 PFLOP/s (bf16) with **8 TB/s** memory bandwidth. **Roofline analysis** tells you whether you're compute-bound or memory-bound.
		- **Kernels**: a kernel is a function that runs on the GPU; each PyTorch primitive launches a standard one. Core principle: **organize computation to minimize data movement**. Naive = read HBM, compute A, write HBM, read HBM, compute B, write HBM; **fused** = read HBM, compute A and B, write HBM. Strategies: operator fusion (matmul + activation), tiling (**FlashAttention**). Written in CUDA/**Triton**/CUTLASS/ThunderKittens.
		- **Parallelism**: with 1024 GPUs, inter-GPU data movement is even slower, but the same minimize-data-movement principle holds. Use collective ops (gather, reduce, all-reduce); shard parameters/activations/gradients/optimizer states; split computation via **{data, tensor, pipeline, sequence, expert} parallelism**.
		- **Inference** (needed for RL, test-time compute, and evaluation) has two phases: **prefill** (tokens given, process all at once → **compute-bound**) and **decode** (one token at a time → **memory-bound**). Speedups: cheaper model (pruning, quantization, distillation), **speculative decoding** (cheap draft model proposes, full model scores in parallel — *exact* decoding), and systems work (fused kernels, continuous batching).
	- ### Scaling laws
		- Setting: given 1e25 FLOPs, what hyperparameters? Tuning at full scale is too expensive.
		- **Key conceptual shift**: instead of a single scale, think of a **scaling recipe** (FLOPs → hyperparameters). Run experiments at smaller scales (e.g. up to 1e24 FLOPs), fit a scaling law, then **predict the loss at the target scale before running it**.
		- Scaling laws **don't happen automatically** — they require careful construction of the recipe. Parameterize the model to get **hyperparameter transfer** (muP). **Predictability is at least as important as optimality.**
		- Given $C = 6ND$: bigger model (N) or more tokens (D)? Compute-optimal answer via **ISOFLOP curves** → **D ≈ 20N** (a 70B model → ~1.4T tokens). **Caveat**: this ignores **inference costs** (in practice you want a smaller model).
	- ### Data
		- **Evaluation** has two purposes: **internal** (guide development — smoothness across scales, *relative* performance matters) and **external** (measure absolute quality of a real use case — *ecological validity* matters). Perplexity should ideally run on **private documents not on the Internet** to avoid contamination. Advanced benchmarks: GPQA, HLE, SWE-Bench, Terminal-Bench. LMs are general purpose → need a **diverse** evaluation set.
		- **Curation**: "data does not just fall from the sky" — webpages, books, arXiv, GitHub. Legal questions: fair use for copyrighted data, or licensing (e.g. Google with Reddit data). Raw data is HTML/PDF/directories, not text.
		- **Processing**: transformation (HTML/PDF → text, extract main content), filtering (quality + harmful content via classifiers), **deduplication** (saves compute, avoids memorization; Bloom filters or MinHash), **data mixing** (how much to up/down-weight each source), and rewriting/**synthetic data**.
		- **Three types of data**: *pretraining* (large and diverse), *mid-training* (high quality, including long-context), *post-training* (SFT: conversations, agentic traces with tool calling).
	- ### Alignment
		- After full supervision (next-token prediction), improve further via **weak supervision** — useful **when it is easier to critique than to generate**.
		- Basic template: (1) generate responses, (2) score with {human, verifier, LM judge}, (3) update the model to prefer better responses.
		- Algorithms: **PPO** (RL), **DPO** (preference data, simpler), **GRPO** (removes the value function).
		- Challenges: RL algorithms are unstable and hard to tune; at scale needs new infrastructure (inference with async rollouts); constant tradeoff between **systems efficiency and on-policyness**.

- ## Unit 1: Tokenization (the actual technical content)
	- **What are the atoms the model operates on?** Formally a tokenizer converts between raw inputs (bytes) and sequences of integers (tokens), via `encode` / `decode`, which must **round-trip**.
	- **The efficiency lens** — why tokenization exists at all:
		- **Reduce context length** (1000 bytes → ~250 tokens). Crucial because **attention is quadratic** in sequence length.
		- **Adaptive computation**: more modeling capacity on interesting parts of the input.
		- Key metric: **compression ratio** = UTF-8 bytes per token. Higher = shorter sequences. You can raise it by increasing **vocabulary size** — but that leads to **sparsity**.
	- ### The four tokenizers, compared
	  | Tokenizer | Vocab size | Compression ratio | Verdict |
	  | --- | --- | --- | --- |
	  | **Character** (Unicode code points via `ord`/`chr`) | ~150K Unicode chars | low | **Worst of both worlds** — huge vocab, and many chars are very rare (e.g. 🌍 = 127757), so the vocabulary is used inefficiently |
	  | **Byte** (UTF-8, ints 0–255) | **256** (nice and small) | **exactly 1** (terrible) | Sequences far too long — untenable given quadratic attention |
	  | **Word** (regex split, e.g. `\w+|.`) | "# distinct chunks in training data" — can be **huge** and not fixed | good | Tokens are meaningful, but many words are rare, no fixed vocab size, and unseen words need an ugly **UNK** token that messes up perplexity |
	  | **BPE** (Byte-Pair Encoding) | tunable | good | **The effective, data-driven heuristic** — the practical answer |
	- ### BPE
		- History: introduced by **Philip Gage in 1994 for data compression**, adapted to NLP for neural machine translation (Sennrich+ 2015), then used by **GPT-2** (Radford+ 2019). Before that, papers had been word-based.
		- **Basic idea**: *train* the tokenizer on raw text to construct a vocabulary tailored to the data — common byte sequences become a single token, rare sequences stay as many tokens.
		- **Sketch**: start with each byte as a token, then successively **merge the most common pair of adjacent tokens**. Training loop: count adjacent pairs → take the `max` by count → assign `new_index = 256 + i` → record the merge and `vocab[new] = vocab[a] + vocab[b]` → apply the merge to the sequence. Repeat `num_merges` times.
		- Observations from playing with a real tokenizer (GPT-5 / `tiktoken` `o200k_base`): a word and its **preceding space** are part of the same token (`" world"`); a word at the **beginning vs middle** of text tokenizes differently (`"hello hello"`); **numbers** are split every few digits.
		- Assignment 1 goes beyond the naive version: only loop over **merges that matter** (not all merges), detect/preserve **special tokens** (`<|endoftext|>`), use **pre-tokenization** (the GPT-2 regex), and make it fast.
		- Deep dive on the training-vs-encoding asymmetry: [[2026-07-tiktoken-bpe-train-encode-decode]]
	- ### The dream: tokenizer-free models
		- Architectures operating **directly on bytes** (Xue+ 2021, Yu+ 2023, Pagnoni+ 2024, Deiseroth+ 2024, Hwang+ 2025) — "promising, but have not yet been scaled up to the frontier."
		- Whatever replaces tokenization must still satisfy: (1) the model should operate on **chunks/abstractions** of the sequence (text, video, DNA), and (2) chunks should be **variable**, so more capacity goes to interesting chunks.
		- Summary from the lecture: character-, byte-, and word-based tokenization are all **highly suboptimal**; BPE is an effective data-driven heuristic; tokenization is a **separate step** — maybe one day it'll be end-to-end from bytes.

- ## Takeaways
	- The whole course collapses to one equation — `accuracy = efficiency × resources` — and efficiency matters *more* as scale grows.
	- What survives scale transfer is **mechanics and mindset**, not intuitions. Design your learning (and your evals) accordingly.
	- Tokenization is not a linguistic choice; it's a **compute-efficiency** choice forced by quadratic attention, which is why the elegant answer (raw bytes) loses to the pragmatic one (BPE).
	- Related: [[2026-05-inference-engineering]], [[2026-05-llm-hw-sw-stack]], [[2026-07-controlling-reasoning-effort-in-llms]], [[2026-06-gpu-connectivity-solutions]]
	- Next lecture: **resource accounting**.
