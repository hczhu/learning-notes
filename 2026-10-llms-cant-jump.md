file-created-at:: 2026-10-25

- **Source**: Tom Zahavy, Google DeepMind, *LLMs can’t jump*, January 27, 2026. Position paper; supplied PDF: `llms-cant-jump.pdf`.
- **One-liner**: Deriving consequences from given premises is different from inventing better premises; the paper proposes counterfactual world models as a bridge to scientific invention.
- **Evidence status**: A historical and conceptual argument—not an experiment or impossibility proof establishing that LLMs cannot invent.

- ## Three kinds of reasoning

	- The paper adopts **Peirce's (1934) Rule–Case–Result framework**: a **Rule** is like a function definition, a **Case** is its input, and a **Result** is its return value. Each mode starts with a different pair and infers the missing element.
	- | Mode | Inference pattern | Meaning and example |
	  | --- | --- | --- |
	  | Deduction | Rule + Case → Result | Applies a known rule to an input; for example, executing code to determine its output. |
	  | Induction | Case + Result → Rule | Generalizes a rule from observed input–output pairs, gaining support from repeated observations; for example, generating a function that satisfies unit tests. |
	  | Abduction | Rule + Result → Case | Infers a plausible case that explains an observed result; the paper also includes proposing a new rule to explain a surprising result. |
	- **Limits**: deduction preserves truth only with true premises and sound inference. Induction remains uncertain beyond the observed cases; passing tests does not prove general correctness or causality. Abduction proposes an explanation that still needs testing—it is not a guaranteed inverse of deduction, since several causes may explain the same result.
	- The author's discovery cycle: **experience → conceptual jump → axioms → deductions → empirical checking**. His central claim is that today's language-based systems lack the grounding needed for the jump to new physical premises.

- ## What the Einstein example illustrates

	- The author interprets general relativity as a response to conceptual problems, not simply a better fit to a large new dataset. Newtonian gravity was broadly successful; the known Mercury anomaly alone did not specify a replacement theory.
	- Thought experiments supplied a different starting point: a falling observer feels locally weightless, while an accelerated elevator suggests the **local equivalence** of gravity and acceleration.
	- Turning that intuition into equations required deduction, but also revising assumptions after unsuccessful approaches. Faster theorem proving would not automatically identify which premises to abandon.
	- **Compression is not the whole story, in the author's view**: optimizing fit or simplicity within an existing representation can favor patching an old theory. Invention may first require new concepts and a larger hypothesis space.

- ## Proposed bridge: think by intervening

	- **Manipulative abduction** means exploring explanatory ideas by manipulating a mental or computational model, rather than only describing observations.
	- Proposed workflow: **change a simulated situation → inspect its consequences → formulate a principle → derive predictions → test against reality**.
	- Passive video prediction is insufficient for this proposal. The system should support interventions—such as cutting an elevator cable—and produce physically consistent consequences, not merely plausible-looking scenes.
	- Physical priors help constrain candidate explanations. The hard step remains translating simulated experience into useful formal axioms; the paper does not demonstrate a system that completes this loop.

- ## Key takeaways and limits

	- **Evaluate discovery separately from problem-solving**: distinguish generating a hypothesis, deriving its consequences, and validating it. Optimizing within fixed rules does not by itself demonstrate the ability to replace those rules.
	- **Do not read the title as a proven universal limit**: the paper offers no new benchmark establishing LLM incapability, and does not show that grounded world models are necessary or sufficient for invention.
	- **Scope matters**: the grounding argument focuses on physical science. Mathematics and computer science would require exploration of abstract structures, not necessarily physical embodiment.
	- **Practical synthesis**: ask whether an AI can revise assumptions and propose tests that distinguish competing explanations. A simulator can inherit mistaken assumptions, so internal consistency must not be confused with empirical truth.

- **Related**: [[2026-10-general-game-playing]] — learning to operate under supplied rules versus inventing new explanatory premises.
