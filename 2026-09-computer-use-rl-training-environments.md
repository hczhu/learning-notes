file-created-at:: 2026-09-18

- **Source**: Excerpt, §1.3 "Computer use training" (with Figure 3, "Overview of a computer-use training workflow"). *(No author or publication byline in the text provided.)*
- **One-liner**: When OpenAI buys tens of thousands of Mac Minis and Mac Studios for reinforcement learning, the Macs are **not** training anything — they are the **environment**. The policy lives on GPUs; the Macs supply a real macOS for it to act on, and the loop between them is ordinary RL with the screen as the observation and mouse/keyboard events as the action space.
- ## 1. The claim that makes the rest make sense
	- The reported purchase of **tens of thousands of Mac Minis and Mac Studios** for RL is consistent with the computer-use trend — but the Macs are **not used to literally train the models** (GPUs are better at that).
	- Their job is to **expose macOS during model training so the model can learn to use the operating system and the tools in it**.
	- **The model likely sits on NVIDIA GPUs and is fed via API to the Mac.** For scale on the other side of that boundary: NVIDIA's CEO mentioned **GPT-6 Astra being trained on ~100,000 Grace Blackwell GPUs**.
	- So a single training run spans **two completely different machines with two completely different jobs** — which is the whole idea worth taking away.
- ## 2. The loop
	- 1. **Prompt** the model with a task — *"open an app xyz and do abc."*
	- 2. **Observe**: provide screenshots of the macOS interface (done by the **harness**).
	- 3. **Act**: the LLM predicts **mouse/keyboard actions** — clicks, key presses, scrolling, and so on.
	- 4. **Execute** those actions on the Mac (harness again).
	- 5. **Re-observe**: feed new screenshots of the updated environment.
	- 6. **Repeat 2–5** until the task **succeeds or fails**.
	- 7. **Reward**: use success/failure signals and **verifiers (graders)** as training feedback, including reinforcement learning during post-training — *analogous to regular **RLVR** (Reinforcement Learning with Verifiable Rewards)*.
	- **The shape of it:**
	  ```
	    +--------------------- GPU cluster ----------------------+
	    |   the POLICY: the LLM being trained                    |
	    +---------^-------------------------------+--------------+
	              |                               |
	     screenshot (observation)          click / keypress (action)
	              |                               |
	    +---------+--------------- HARNESS -------v--------------+
	    |   captures screens, transports over API, replays input |
	    +---------^-------------------------------+--------------+
	              |                               |
	    +---------+------------- Mac fleet -------v--------------+
	    |   the ENVIRONMENT: real macOS + real apps              |
	    +--------------------------+-----------------------------+
	                               | terminal state (success/fail)
	                               v
	                       verifier / grader
	                               |
	                             reward  ------>  RL update, on the GPUs
	  ```
- ## 3. It is textbook RL — the vocabulary maps exactly
	- The fastest way to stop finding this exotic is to line it up against the standard RL objects:
	- | RL concept | What it is here |
	  |---|---|
	  | **Environment** | **macOS** running on a physical Mac, with its real apps |
	  | **Agent / policy** | The **LLM**, running on GPUs elsewhere |
	  | **Observation** | A **screenshot** — the state arrives as *pixels*, not as structured API data |
	  | **Action space** | **Mouse and keyboard events**: click, key press, scroll |
	  | **Step** | One observe → act → execute cycle |
	  | **Episode / rollout** | One task attempt, run until success or failure |
	  | **Reward** | Verifier/grader output on the **terminal** state — sparse, at the end |
	  | **Harness** | The plumbing that closes the loop: screen capture, API transport, input replay |
	- **The harness is the load-bearing component** and the thing the excerpt mentions twice. The model never touches the Mac; it emits action *descriptions*, and the harness is what turns them into real input events and turns real pixels back into tokens.
- ## 4. Why a real machine instead of a simulator
	- *(This part is inference from the setup, not stated in the excerpt.)*
	- **You get the reward you can measure, and you learn the environment you were given.** A mocked or simplified GUI would teach the model the mock: its rendering, its timing, its quirks. The point of buying real hardware is **fidelity** — the distribution of window chrome, animation delays, focus behaviour, dialog placement, and app misbehaviour *is* the thing being learned.
	- **macOS only runs on Apple hardware** in practice, so unlike a Linux environment you cannot simply spin up containers on the GPU cluster's own nodes. The fleet is the price of the operating system.
	- **Environment state must be resettable.** Episodes pollute the machine — files created, preferences changed, apps left open — so between rollouts the environment has to return to a clean baseline, or the reward signal drifts. Any large fleet like this implies snapshotting/reimaging machinery behind it.
- ## 5. Why the fleet has to be *that* big
	- *(Also inference, but it follows directly from the two-machine split.)*
	- In math or code RLVR, the verifier is **a program**: it runs in milliseconds, deterministically, on the same commodity CPU as everything else. Rollouts are cheap, so the GPUs stay busy.
	- A GUI episode is **wall-clock bound by a real operating system**: apps launch, animations play, the network is consulted, a human-scale UI takes human-scale time. A single episode is seconds to minutes, and **no amount of GPU capacity makes a Mac open Preview faster**.
	- Therefore the fleet size is set by a **ratio**, not by a compute requirement: *how many environments must run concurrently to keep 100,000 GPUs supplied with rollouts?* When each environment is slow and serial, the answer is **tens of thousands of them**. The Macs are not a compute purchase; they are a **latency-hiding purchase** — buying parallelism to cover a slow resource, which is the same move GPUs make with threads and memory latency ([[2026-09-pmpp-ch1-cpu-vs-gpu-design-philosophies]]).
- ## 6. How this differs from ordinary RLVR
	- | | **Classic RLVR** (math, code) | **Computer-use RL** |
	  |---|---|---|
	  | **Environment** | A checker: SymPy, a compiler, unit tests | A real OS with real applications |
	  | **Observation** | Text | **Screenshots** (pixels) |
	  | **Action** | Emit tokens | Emit **input events** that change external state |
	  | **Verification** | Cheap, deterministic, programmatic | Constructed **graders** over final state; slower, and harder to write |
	  | **Episode cost** | Milliseconds | Seconds to minutes of wall clock |
	  | **Side effects** | None — rerun freely | Real: files written, apps opened, state mutated; needs reset |
	- The **reward is still terminal and sparse** — success or failure of the whole task — which is the property it shares with RLVR and the reason the same post-training machinery applies. (In RLVR the intermediate reasoning trace is not used to update the model either; see [[2026-07-controlling-reasoning-effort-in-llms]].)
- ## 7. Takeaways
	- **"Buying hardware for RL" does not mean buying compute.** The Macs are an *environment purchase*. Separating the policy (GPUs) from the world it acts on (Macs) is the single structural fact, and every other detail follows from it.
	- **Computer use is not a new algorithm — it is a new environment.** The learning machinery is the RLVR post-training already in use; what changed is that the observation became pixels, the action became input events, and the verifier became a grader over real system state.
	- **The harness is where the engineering lives.** Screen capture, action replay, API transport, reset between episodes, grading — none of it is modelling work, and all of it determines whether the loop closes.
	- **Environment throughput, not model size, is the bottleneck of agentic RL.** The scarce resource is *rollouts per second through a slow real-world environment*, which is why the fleet is measured in tens of thousands. Expect this to be the recurring constraint for any RL against real software.
	- Related: [[2026-07-controlling-reasoning-effort-in-llms]], [[2026-04-cs336-lecture-01-overview-tokenization]], [[2026-09-pmpp-ch1-cpu-vs-gpu-design-philosophies]], [[2026-07-dex-ai-coding-context-engineering]]
