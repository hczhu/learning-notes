## Future directions
	- **Memory as a first-class subsystem**
		- CC today exposes the factual tier (CLAUDE.md, auto-memory) and the working tier (the conversation window);
		- **The experiential tier** (accumulated, automatically curated playbooks of strategies learned from past sessions) is the natural next
	- **Observability and silent failure**
		- Industry surveys suggest that the dominant failure mode of deployed agents is not crashes but silent mistakes
		-
		-
- **The core of the system** is a simple while-loop that calls the model, runs tools, and repeats. Most of the code, however, lives in the systems around this loop
	- A permission system with seven modes
	- An ML-based classifier
	- A five-layer compaction pipeline for context management
	- Four extensibility mechanisms (MCP, plugins, skills, and hooks)
	- A subagent delegation and orchestration mechanism,
	- Append-oriented session storage
- ## Five Values and Philosophies
	- **Human Decision Authority** - The human retains ultimate decision authority over what the system does
	- **Safety, Security, and Privacy** - authority is about the human’s power to choose, safety is about the system’s obligation to
	  protect even when that power lapses
	- **Reliable Execution**
		- a three-phase loop that the agent repeats until the task is complete: gather context, take action,
		  and verify results
	- **Capability Amplification** - materially increases what the human can accomplish per unit of effort and cost. materially increases what the human can accomplish per unit of effort and cost.
		- The architecture invests in deterministic infrastructure (context
		  management, tool routing, recovery) rather than decision scaffolding (explicit planners or state graphs), on
		  the premise that increasingly capable models benefit more from a rich operational environment than from
		  frameworks that constrain their choices.
	- **Contextual Adaptability** - The system fits the user’s specific context (their project, tools, conventions, and
	  skill level) and the relationship improves over time
- ## Context Construction and Memory
- Design choices
	- File-based transparency
	- Database-backed retrieval
	- Opaque learned representations
- ### Compaction Pipeline
	- Budget reduction (always active): per-tool-result size limits.
	- Snip (HISTORY_SNIP): lightweight older-history trimming.
	- Microcompact (CACHED_MICROCOMPACT): fine-grained cache-aware compression.
	- Context collapse (CONTEXT_COLLAPSE): read-time virtual projection over history.
	- Auto-compact (enabled by default, can be disabled): full model-generated summary.
- ## The Model and Harness
- The model reasons about what to do; the harness is responsible for executing actions. The model emits **tool_use** (a structured protocol) blocks as part of its response, and the harness parses them, checks permissions, dispatches them to tool implementations, and collects results (query.ts).
- Claude Code uses a single queryLoop() function that executes regardless of
  whether the user is interacting through an interactive terminal, a headless CLI invocation, the Agent SDK,
  or an IDE integration (query.ts).
- Five distinct context-reduction strategies execute before every model call (query.ts), and several other subsystem decisions (lazy loading of instructions, deferred tool schemas, summary-only subagent returns) exist to limit context consumption
- ## Different orchestration patterns
- **ReAct pattern** - the model generates reasoning and tool invocations, the harness executes actions, and results feed the next iteration.
- **Explicit graph-based routing** - control flow is defined as a state machine with typed edge
- **Tree-search methods** - explore multiple action trajectories before committing
- ## Pre-model context shapers
- **Budget reduction** - (applyToolResultBudget()). Enforces per-message size limits on tool results, replacing oversized outputs with content references.
- **Snip** - (snipCompactIfNeeded(), gated by HISTORY_SNIP). A lightweight trim that removes older history segments, returning {messages, tokensFreed, boundaryMessage}.
- **Microcompact** Fine-grained compression that always runs a time-based path and optionally a cache-aware
  path
- **Context collapse** A read-time projection over the conversation history.
- **Auto-compact** The fifth shaper, triggering a full model-generated summary via compactConversation()
  in compact.ts. This function executes PreCompact hooks, creates a summary request using getCompactPrompt(), and calls the model to produce a compressed summary. The result feeds into buildPostCompactMessages() (compact.ts). Auto-compact fires only when the context still exceeds the pressure threshold after all four previous shapers have run.
- ## Extensibility
	- **MCP servers** provide external tool integration
	- **Plugins** package and distribute bundles of components
	- **Skills** inject domain-specific instructions
	- **Hooks** intercept the tool execution lifecycle.
- ## 6 built-in subagents
	- Explore: primarily read/search-oriented investigation, with write and edit tools in its deny-list
	- Plan: creates structured plans; execution proceeds through the standard permission model.
	- General-purpose: broadly capable, used when explicitly requested (note: omitting the type may route to the fork-subagent path instead).
	- Claude Code Guide: onboarding and documentation assistance, with its own permissionMode override.
	- Verification: runs validation checks (test suites, linting)
	- Statusline-setup: specialized for terminal status line configuration.
- ## Multi-agent coordination challenges
- Multi-agent systems differ fundamentally from single-agent systems, with **coordination complexity growing rapidly** as agents are added.
	- Early failure modes observed:
		- Spawning **50 subagents** for simple queries
		- Scouring the web endlessly for **nonexistent sources**
		- **Distracting each other** with excessive status updates between agents
- Since each agent is steered by a prompt, **prompt engineering is the primary lever** for correcting these behaviors.
- ## Diagram
- ![image.png](../assets/image_1777414717188_0.png)
- ![image.png](../assets/image_1777414039562_0.png)
- ![image.png](../assets/image_1777413662369_0.png)
- ![image.png](../assets/image_1777409947731_0.png){:height 435, :width 952}
- ![image.png](../assets/image_1777410749880_0.png)
-