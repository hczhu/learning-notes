file-created-at:: 2026-10-18

- **Source**: Michael Genesereth, *General Game Playing*, Communications of the ACM, April 2026, Vol. 69, No. 4, pp. 72–79. [DOI: 10.1145/3747852](https://doi.org/10.1145/3747852). Based on the supplied photographs.
- **One-liner**: General intelligence requires turning unfamiliar rules into effective behavior; analyzing and restructuring a problem can matter as much as searching harder.

- ## What Makes Game Playing General?

	- A general game player receives a formal description of an unfamiliar game at runtime and must play without human intervention. It cannot depend on a programmer having supplied that game's strategy or evaluation function beforehand.
	- The challenge includes different goals, numbers of players, and competitive or cooperative settings, not merely playing familiar games well. The author argues that this shifts more of the intelligence from the programmer into the program.
	- Game descriptions may use arbitrary or obfuscated names. A system must reason from the rules rather than assume that a familiar label implies familiar behavior.

- ## Rules Become an Executable Model

	- A game can be modeled as states, legal actions, transitions, terminal conditions, and rewards for each player. From its description, the agent must derive all of these, including which moves are legal now and what each move changes.
	- **Game Description Language (GDL)** expresses these relationships with logical facts and rules. This compact representation can describe thousands of states without listing them individually.
	- **Knowing the rules is not knowing how to win.** The agent can construct a game tree from the model, but large trees and move deadlines make exhaustive search impractical. It needs ways to evaluate unfinished positions or simplify the problem.

- ## Three Ways to Choose Better Moves

	- | Approach | Main idea | Important limitation |
	  | --- | --- | --- |
	  | General heuristics | Estimate value using features such as mobility, proximity to a goal, or restricting opponents. | A useful pattern in one game can be harmful in another; more legal moves is not universally better. |
	  | Monte Carlo search / tree search (MCTS) | Estimate a position's value through simulated continuations; MCTS guides exploration using results accumulated so far. | Sampling provides estimates, not guaranteed optimal play, and must fit the computation budget. |
	  | Metagaming | Analyze the rules before play: find independent parts, reformulate the description, or generate specialized procedures. | The analysis itself costs time and useful structure is not always easy to discover. |

- ## Decomposition Can Beat Brute Force

	- **Hodgepodge** combines chess and Othello on separate boards. Winning requires both, but moves in one board do not affect the other, making separate analysis possible.
	- If independent subgames have branching factors `a` and `b`, joint search to depth `n` can involve approximately `(a × b)^n` states; separate searches involve approximately `a^n + b^n`. The benefit depends on genuine independence, not simply splitting the board arbitrarily.
	- **Key insight**: sometimes the work needed to discover a simplification scales with the compact game description rather than the enormous expanded game tree. Spending time understanding the problem can save much more time during play.

- ## Broader Lessons for AI

	- **Knowledge representation and reasoning remain central.** Explicit rules let a system infer consequences and construct task-specific methods instead of relying entirely on memorized patterns.
	- **Treat the article's ML/LLM claims as scoped claims.** The author reports that no machine-learning program had won an official GGP competition and argues that time limits and obfuscated descriptions challenge LLM-based play. This is not a proof that learning or hybrid approaches cannot succeed.
	- **Practical takeaway**: for an unfamiliar task, first identify its rules, goals, dependencies, and resource limits; then choose between heuristics, simulation, and structural simplification. Do not assume a familiar strategy fits a new objective.
	- **The larger ambition** is declarative problem solving: users describe what they want, while the system determines how to achieve it. GGP is a controlled laboratory for that capability, not just a collection of games.
