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

- ## ARC-AGI: a logical jump, not the whole discovery loop

	- **Task described in the paper (§5)**: infer a hidden transformation from sparse examples—typically **2–5 input–output grid pairs**—and apply it to a new input. The solver receives examples, not an explicit rule to execute.
	- **Why the author calls this abduction**: the solver must propose a plausible explanation that accounts for the examples. This differs from deductively applying a supplied rule; the author argues that such sparse evidence also challenges a purely statistical account of discovery.
	- **What ARC captures versus misses, in this argument**: it tests explaining supplied observations, but not the preceding step of inventing a physical situation, simulating an intervention, and using the imagined experience to generate new explanatory principles. Einstein's thought experiments combine **simulation to produce an imagined observation** with **abduction to explain it**.
	- **Related distinction**: generating variations of chess or mathematics problems at test time still operates within established rules. The paper contrasts this with revising the physical axioms themselves, as in Einstein's elevator thought experiment.
	- **Interpretation limit**: this is the author's account of ARC's relevance, not a benchmark result demonstrating how a model reasons. Sparse examples alone do not rule out learned priors or statistical methods; success on grid tasks does not by itself establish physically grounded scientific invention.

- ## Proposed bridge: think by intervening

	- **Manipulative abduction** means exploring explanatory ideas by manipulating a mental or computational model, rather than only describing observations.
	- Proposed workflow: **change a simulated situation → inspect its consequences → formulate a principle → derive predictions → test against reality**.
	- Passive video prediction is insufficient for this proposal. The system should support interventions—such as cutting an elevator cable—and produce physically consistent consequences, not merely plausible-looking scenes.
	- Physical priors help constrain candidate explanations. The hard step remains translating simulated experience into useful formal axioms; the paper does not demonstrate a system that completes this loop.

- ## Beyond language models: video generation and interactive world models

	- **Grounding problem**: the author uses the “Chinese Room” analogy to distinguish manipulating descriptions of physics from connecting symbols to perceptual experience and action. This is a philosophical framing, not an experimental proof that language models cannot learn physical representations.
	- | Model or proposed direction | Capability emphasized in the paper | Remaining gap |
	  | --- | --- | --- |
	  | Video generators, illustrated by Veo | Produce plausible visual continuations, such as an unsupported apple falling. | The author argues that plausible motion can reflect training correlations; appearance alone does not establish reliable causal physics or intervention capability. |
	  | Action-controllable world models, illustrated by Genie (Bruce et al., 2024) | Learn an action space through which an agent can influence the generated environment. | Interactivity enables “thinking by doing,” but does not itself establish physical consistency or scientific invention. |
	  | Proposed future world models | Support counterfactual interventions through a consistent internal representation of physical dynamics, rather than merely plausible pixels. | Turning simulated outcomes into new axioms—and validating those axioms against reality—remains a research proposal. |
	- **Counterfactual intervention**: watching an elevator fall is different from controlling a simulation to cut its cable and investigate what changes. The proposed synthetic laboratory would let an agent generate observations through interventions, then seek explanations for them.
	- **Priors direct exploration**: Einstein's physical intuition helped narrow the search for axioms. The author proposes systems with structured expectations that guide which hypotheses and simulations to try, extending the idea beyond physics to pre-symbolic intuitions in other fields. These priors guide inquiry; they are not evidence that a preferred explanation is true.
	- **Boundary of the claim**: this compares capabilities discussed in the January 2026 paper, not current product versions or mutually exclusive architecture classes. The key distinction is **prediction versus controllable, grounded experimentation**, not simply “LLM versus non-LLM.”

- ## Key takeaways and limits

	- **Evaluate discovery separately from problem-solving**: distinguish generating a hypothesis, deriving its consequences, and validating it. Optimizing within fixed rules does not by itself demonstrate the ability to replace those rules.
	- **Do not read the title as a proven universal limit**: the paper offers no new benchmark establishing LLM incapability, and does not show that grounded world models are necessary or sufficient for invention.
	- **Scope matters**: the grounding argument focuses on physical science. Mathematics and computer science would require exploration of abstract structures, not necessarily physical embodiment.
	- **Practical synthesis**: ask whether an AI can revise assumptions and propose tests that distinguish competing explanations. A simulator can inherit mistaken assumptions, so internal consistency must not be confused with empirical truth.

- **Related**: [[2026-10-general-game-playing]] — learning to operate under supplied rules versus inventing new explanatory premises.
