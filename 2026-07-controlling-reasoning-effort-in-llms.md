- **Source**: Sebastian Raschka, PhD, "Controlling Reasoning Effort in LLMs — How LLMs Learn Low-, Medium-, and High-Effort Reasoning Modes," *Ahead of AI* (Substack), 18 Jul 2026.
- **One-liner**: A reasoning model's "effort" or "thinking" setting is not a prompt-engineering trick — it is a **trained behavior**. Effort maps to how many reasoning tokens the model spends (a form of inference-time scaling), taught via effort-conditioned SFT and/or token-cost-shaped RL, and selected at inference through system prompts or chat-template toggles.

- ## 1. What reasoning models are
	- A "reasoning model" outputs an **intermediate reasoning trace** (a step-by-step working-out) before the final answer. It doesn't "think" like a human — the term is not literal.
	- vs. a conventional LLM: a reasoning model shows intermediate reasoning, can **self-correct and backtrack**, and is easier to verify/debug (at the cost of more tokens).

- ## 2. Training vs. inference scaling
	- Two levers improve reasoning performance: **training-time compute scaling** and **inference-time compute scaling**.
	- **Training (RLVR)**: DeepSeek-R1 popularized **reinforcement learning with verifiable rewards** — a 0/1 reward signal on verifiable domains (math via SymPy/WolframAlpha; code via compilers/unit tests/LeetCode). Reward = `R_accuracy + R_format`; the **intermediate reasoning trace itself is not used** to update the model (DeepSeek found it unhelpful) — only final-answer correctness + format.
		- **"Aha" moments**: training on output rewards alone was sufficient for models to *learn* to write intermediate reasoning, backtrack, and self-correct.
		- **Pure RL works**: DeepSeek-R1-Zero applied RLVR directly to the base model with **no SFT** (weaker than R1, but proved RLVR alone teaches reasoning). Full R1 pipeline is multi-stage (cold-start SFT → RL → distillation into smaller Qwen/Llama variants).
	- **Inference scaling**: reasoning models already spend more tokens at inference. Additional techniques: **self-consistency** (majority vote over multiple samples), **self-refinement** (sequential iterations). DeepSeekMath-V2 stacked extreme inference-scaling on a math reasoning model for SOTA olympiad performance.

- ## 3. Think tokens are cosmetic
	- `<think>`/`</think>` tags **do not make the model reason** and aren't required for good performance — you could train the same model without them and reach similar benchmarks. The literal strings aren't special either; any delimiters work.
	- Their only purpose: **mark where the reasoning trace begins/ends** so the pipeline or UI can separate it from the final answer (and optionally hide it). Implemented via a **format reward** during RLVR that encourages wrapping reasoning in the tags.

- ## 4. Reasoning on/off switches
	- 1st-gen reasoning models (e.g. R1) were **dedicated** — always verbose, even for "what is 1+1?", with no off switch. Later models (Qwen3, etc.) made it **toggleable** — same model behaves as instruction-tuned OR reasoning on demand. ("Thinking mode" and "reasoning mode" are the same thing.)
	- **Qwen3 mechanism**: tokenizer flag `enable_thinking=True/False`. Setting `False` prefills an **empty `<think></think>` block** at the start of the assistant response so the model proceeds straight to the answer.
	- **How it's trained** ("Thinking Mode Fusion", post-RL SFT stage): the model sees both
		- `/think`: `<think>{reasoning}</think>{answer}`
		- `/no_think`: `<think></think>{answer}`
		- `/think` is default (can be omitted); a later general-RL stage reinforces both. The `/think`–`/no_think` flags are a **"soft" switch**; force-adding the empty tags (`enable_thinking=False`) is a **"hard" switch**.

- ## 5. How "reasoning effort" settings work
	- Effort is a generalization of the on/off toggle to multiple levels. GPT-5.6 exposes ~6 levels (Light → Medium → High → Extra High → Max → Ultra); OpenAI doesn't disclose the implementation.
	- **Effort ↔ length ↔ accuracy**: evidence from open-weight `gpt-oss` — effort is set via a **system prompt** ("Reasoning: high") inserted by the chat template into the same model's weights. Higher effort → more CoT+answer tokens → higher accuracy (AIME, GPQA). Effort is directly correlated with token usage, which correlates with accuracy — but with **diminishing returns / saturation** (extra reasoning budget becomes uneconomical; visible on the GPT-5.6 Sol cost curve).
	- **Two ways to train effort control**:
		- **RLVR with effort-dependent length penalty**: high penalty when system prompt says "Reasoning: low", mild/none when "high".
		- **Effort-conditioned SFT after RLVR**: pair "Reasoning: low/high" prompts with short/long target traces.
		- (Likely combined in practice.)
	- **Inkling case study** (Thinking Machines Lab): uses a **continuous effort value 0.0–1.0** (not ordinal labels). During large-scale async RL (>30M rollouts): (1) put desired effort in the system message, (2) adjust per-token cost. Conceptual reward:
	  $$
	  R(e) = R_{\text{task}} - \lambda(e)\, N_{\text{tokens}}
	  $$
	  where low effort → larger per-token cost (shorter traces), high effort → smaller cost (more tokens). Effort conditioning lives in the **Reasoning RL** stage.
	- **Two orthogonal knobs**: model size (training scaling) and reasoning effort (inference scaling). Their cost/accuracy curves **overlap** — a smaller model at high effort can match a larger model at low effort. Best combo depends on target accuracy/cost/latency. (In the GPT-5.6 Luna/Terra/Sol chart: moving *along* a curve = inference scaling; moving *across* curves = model/training scaling.)

- ## 6. How six flagship open-weight models implement effort
	- | Model | Effort control | Disclosed training mechanism | Inference control |
	  | --- | --- | --- | --- |
	  | **DeepSeek V4** | Non-think, High, Max | Separate **effort specialists** via mode-specific SFT + GRPO with per-mode context windows & length penalties, then on-policy distillation into one checkpoint | Response format + a "Max" system instruction (longer context, smaller length penalty) |
	  | **Nemotron 3 Ultra** | Off, medium, regular | Teacher-generated medium traces (from GPT-OSS-120B), random-budget truncation, length-adjusted RLVR (~2.5% of RL prompts medium-effort) | Learned mode + optional **hard token budget** (client closes `</think>` externally if not emitted) |
	  | **Kimi K2.5** | Implicit token-efficient reasoning | **"Toggle"**: alternates budgeted & unconstrained RL phases using per-problem budgets (p-th percentile of correct-rollout lengths; budget activated only after accuracy passes a threshold) | Thinking/instant modes exist, but Toggle is not a separate selector; final checkpoint is unified |
	  | **GLM-5** | Thinking on/off per turn + interleaved & preserved thinking | Multi-task SFT + updated chat template; sequential reasoning/agentic/general RL; on-policy cross-stage distillation | Binary per-turn switch (`<\|assistant\|><think>` vs `</think>`), auto reasoning before tool calls |
	  | **Qwen3** | No-think, think, token budget | Thinking Mode Fusion SFT + general RL | `/think`, `/no_think`, and inference-time truncation (partial-reasoning behavior emerged, not explicitly trained) |
	  | **Inkling** | Continuous scalar 0.0–1.0 | Effort-conditioned RL with effort-dependent token cost | Effort value in system message |
	- **Shared framework across all six**: (1) introduce effort modes via SFT + chat template; (2) a **mode-conditioned RL stage** where context windows/length penalties vary with requested effort (DeepSeek V4, Nemotron, Inkling); (3) **robustness under hard budgets** — train on randomly truncated traces (Nemotron), continue from a forcibly stopped span (Qwen3), or alternate budgeted/unconstrained RL (Kimi) so quality survives when reasoning is cut short.
	- **Kimi Toggle result**: ~25–30% fewer generated tokens with little benchmark change; transfers from math/code RL to GPQA/MMLU-Pro.

- ## 7. Conclusion & outlook
	- Same effort *labels* can be backed by very different mechanisms (separate specialists, mixed SFT data, mode-conditioned rewards, hard token budgets, or combinations). No clear "best" — reports omit details for controlled comparison, and the right method may differ for an interactive assistant vs. a long-running coding agent.
	- The **holy grail is automatic effort selection** (like GPT-5's since-removed "Auto" mode — hard to get right).
	- Raschka's forecast: reasoning effort will **remain an explicit model input** (usually via system prompt), but agent harnesses / internal routers will increasingly **infer the appropriate mode & budget from task state and available resources automatically**, while still allowing user override (for latency/cost/max-performance).
	- Related: [[2026-05-inference-engineering]], [[2026-01-annotated-jepa-ijepa]]
