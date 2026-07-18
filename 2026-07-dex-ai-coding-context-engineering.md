- **Source**: Podcast summary of Dex (Dexter Horthy, HumanLayer) — author of "[12-Factor Agents: Principles for building reliable LLM applications](https://github.com/humanlayer/12-factor-agents)." Notes on AI coding, context engineering, and running a "software factory."
- **One-liner**: Never ship code no human has read; keep humans in the loop where leverage is highest (design/architecture), and treat the context window — not the model — as the thing you actually engineer.

- ## Origin of 12-Factor Agents
	- Around **Aug 2024**, Dex started building AI agents when the common approach was frameworks like **LangChain** and **CrewAI**.
	- He talked with **~100 "real" AI engineers** doing tangible work (e.g. $100K+ contracts shipping AI solutions inside enterprises). The recurring pattern: they **tried the frameworks and discarded them** in favor of building their own pipelines.
	- Those conversations became the book **12-Factor Agents**.

- ## The cautionary tale — shipping unread code
	- **Jul 2025**: Dex experimented with letting the model write code while **humans reviewed nothing**.
	- **Four months later** they shut it down and threw the whole system out. Production broke, and no amount of prompting **Opus 4.1** could find the root cause.
	- It took **days** of wading through spaghetti code to find a **primary key wrongly routed through the entire codebase**. After fixing it, **three weeks** to re-onboard to a codebase no human had ever read.
	- Today he thinks this failure would develop in **far less than four months**, because newer models produce code much faster than a year ago. **Lesson: shipping unread code spells disaster within months.**

- ## Why coding models degrade codebases over time
	- Dex's hypothesis: today's coding models are optimized for **SWE-bench-style benchmarks**, which reward **reproducing a known fix** in codebases like Django but **cannot measure bad architecture / bad program design**.
	- The core problem: the **cost function of bad architecture cannot be evaluated by a unit test.** So models get good at localized fixes while silently worsening structure.
	- His proposed better eval: have a model **build 20 features in a row** in a codebase **without knowing what's coming next** — measuring compounding design quality, not one-shot patches.

- ## Context engineering
	- ### Find where the "dumb zone" begins
		- Rule of thumb: **the less of the context window you use, the better the outcome.** Attention is **quadratic** — more context = more compute to process it all.
		- Dex's heuristics: for a **1M-context** model, push to ~**300–400K** when it feels right; for **smaller models**, stop around **100K**.
		- The **"dumb zone"** is where performance degrades past this limit and the model starts "doing increasingly stupid things like deleting your .env file."
	- ### A bigger context window ≠ a smarter model
		- Intelligence lies in the model's ability to **use** the tokens — deciding which parts of context are relevant for the next decision. You have to develop a *feel* for how much context usage makes sense, and experiment when you hit the dumb zone.
	- ### The only four things that matter in the context window
		- **Size**: bigger → more room before the dumb zone.
		- **Information quality**: once something is in context, every subsequent turn treats it as **fact** — which is why errors **compound**.
		- **Missing information**: gaps make the agent **guess**, worsening outcomes.
		- **Trajectory**: models are **autoregressive**, predicting the next message from prior ones. "Trajectory poisoning" = the agent locks into a pattern you don't want → time to start over.

- ## Techniques & workflows
	- ### Frequent, intentional compaction
		- Take a long/noisy context, **compress it into a Markdown doc**, then start a **fresh session** pointed at that "compressed context."
		- Example multi-session pipeline:
			- Session 1: reads a ton of code (filling context while still in the "smart zone") → emits a **research document**.
			- Session 2: takes tickets describing the work → turns them into a **design document**.
			- Session 3: takes both docs → creates a **plan**.
			- **Human in the loop where it matters**: reviewing the **design document and architecture** (Dex finds models weak here).
	- ### "You're completely right!" = start a new session
		- Phrases like "You're completely right!" or "You're right to push back on that" signal a **trajectory-poisoned** session — continuing wastes time and tokens.
		- The autoregressive failure loop: model makes a mistake → user "yells" → model keeps erroring → user "yells" → the model calculates that the next most probable message is **another mistake**.
	- ### Slow loops ("loop engineering")
		- HumanLayer started with a **nightly automation**: an agent fixes one thing and opens a PR overnight, so the team wakes up to a PR ready to merge.
		- Now **four agents open four PRs by morning**, focused on **code-quality improvements** — and a **person reads all of them before merging**.

- ## Cost & model-selection philosophy
	- **Don't optimize LLM usage until business is booming / at massive scale / costs are high.** Always start building with the **smartest available model**, because **engineering time is almost always the bottleneck**.
	- Only optimize context/model usage at real scale — that's when it's worth using something like **GPT-OSS-120B (~1/1000th the cost of Opus)** for the **simpler steps** of a pipeline.
	- **"Token harder" vs. "token smarter"**: "token harder" = maxing out Claude subscriptions (his 'Hyper Engineering' group chat). "token smarter" = maximum value from AI while keeping control — harder to pull off, and the goal.

- ## Three ways to run a "software factory"
	- 1. **"Turn the lights off"** — all-in agentic coding, no code review, pray the AI doesn't create too much slop. **Dex tried this and failed.**
	- 2. **Read and review all AI-generated code** — slows things to human speed; expect a **30–50% productivity lift** over pre-AI engineering.
	- 3. **Find leverage, keep people in the loop (his recommendation)** — find where **1 hour of planning saves ~4 hours of implementation** (fewer bugs). Invest in **design, architecture, key decisions**; then let the agent generate code without insisting on reviewing all of it. He believes this moves **2–3× faster** than hand-writing all code.

- ## Takeaways
	- The durable constraint isn't model capability — it's **human comprehension of the codebase**. Optimize for keeping a human able to understand and steer the architecture.
	- Engineer the **context window** (size, quality, completeness, trajectory), not just the prompt; know your model's dumb zone and compact aggressively.
	- Put humans where leverage is highest (planning/design), automate where it isn't (implementation, quality PRs), and never confuse "token harder" with progress.
	- Related: [[2026-06-dive-into-claude-code]], [[2026-07-controlling-reasoning-effort-in-llms]], [[2026-05-inference-engineering]]
