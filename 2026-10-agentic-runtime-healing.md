file-created-at:: 2026-10-17

- **Source**: Zhensu Sun, Haotian Zhu, Bowen Xu, Xiaoning Du, Li Li, and David Lo, *Toward Agentic Runtime Healing*, Communications of the ACM, September 25, 2026. Based on the supplied PDF. [Research artifacts](https://github.com/v587su/Healer).
- **One-liner**: LLMs can generate context-specific recovery code for unexpected runtime errors, but keeping a program running is much easier than preserving its intended behavior.

- ## What Runtime Healing Changes

	- **Repair the current execution, not the source code.** Healer changes runtime state, such as variable values or environment settings, so execution can resume. The underlying bug remains and still needs a permanent fix.
	- **Generate a recovery strategy instead of selecting only predefined rules.** Restarting or retrying can address transient failures, but repeating identical logic and inputs will not fix deterministic errors such as an invalid index or undefined variable.
	- **Workflow**: unhandled error → collect source, error message, location, and pre-error state → ask the LLM for healing code → execute it with that state → transfer the resulting state back and resume after the failing line. If healing raises an error, the prototype terminates and reports it.
	- **Example**: when a log collector's directory creation fails because the directory already exists, generated recovery code chooses a different output directory rather than abandoning the collected logs.

- ## What the Experiments Show

	- The Python proof of concept evaluates error-triggering code/test pairs: 1,185 across CodeNet, HumanEval+, MBPP+, and DebugBench, plus 47 LiveCodeBench control samples.
	- | Model, without task-specific fine-tuning | Execution proceeds | Correct test result |
	  | --- | --- | --- |
	  | GPT-4 | 73.0% | 38.7% |
	  | GPT-3.5 | 48.3% | 23.5% |
	  | CodeQwen | 35.1% | 13.0% |
	- **Availability is not correctness.** Correct results are a subset of executions that proceed, not an additional category. Simply suppressing errors produced correct results in only 4.3% of the authors' baseline experiment.
	- **Specialization helps.** Fine-tuning on 586 successful recovery examples, with CodeNet held out for evaluation, raised GPT-3.5's correct cases from 71 to 90 out of 228, close to GPT-4's 96. CodeQwen improved from 38 to 63. These are benchmark gains, not evidence of production safety.
	- **Reporting caveat**: the introduction gives slightly different GPT-4 figures, 72.8%/39.6%; the table above follows the evaluation section's all-benchmark summary. Test-case success also does not establish correctness for every input or side effect.

- ## Trustworthiness Is the Main Adoption Barrier

	- Generated code can skip essential logic, omit returns, or silently substitute incorrect values. A program that survives may now produce misleading outputs.
	- In a manual review of 204 incorrect-but-executable GPT-4 healing snippets from DebugBench, the authors report that 75% exhibited recognizable behavior patterns. **Recognizable does not mean safe**: value changes or imports can be detectable yet inappropriate for the application.
	- Proposed controls include static checks and behavior allowlists, testing or verification, isolated execution, and restricting the LLM to vetted recovery functions rather than arbitrary code. These are proposed safeguards, not a demonstrated safety guarantee.

- ## Engineering Takeaways

	- Keep existing handlers and rollback paths; use agentic healing as a constrained fallback, not their replacement.
	- Define where intervention is acceptable, starting with low-risk components. Log the error and recovery for later debugging, and make uncertain outputs visible to users.
	- Retrieve only relevant code and state in large systems. Remote models require network access; local models require inference resources. The experiments demonstrate Python feasibility, not turnkey support for other runtimes.
	- **Practical inference**: evaluate recovery by application invariants and acceptable side effects, not just absence of exceptions. If a proposed repair cannot be validated within those boundaries, prefer the established failure or rollback path.
