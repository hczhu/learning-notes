- **Source**: "Processors are getting wider," blog post, 1 September 2025 (~3 min read). *(No author byline in the text provided.)*
- **One-liner**: Clock frequency stopped scaling around 5 GHz, so single-core speed now comes from **width** — retiring more instructions per cycle — which means your code already runs in parallel under the hood whether or not you asked for it.
- ## 1. The frequency ceiling
	- A processor executes instructions against a clock: **4 GHz = 4 billion cycles per second**.
	- Raising that clock is hard. **Much beyond 5 GHz and the processor overheats or otherwise fails.** So the cycles-per-second term is effectively pinned.
	- With frequency fixed, the only remaining lever in a single core is **instructions per cycle (IPC)**.
- ## 2. Superscalar execution — the width lever
	- Modern processors execute **multiple instructions simultaneously**; this is **superscalar execution**.
	- **Most processors handle 4 instructions per cycle or more.** A recent Apple processor **easily sustains over 8 per cycle**.
	- But IPC is **not a fixed property of the chip** — it depends on the instruction mix. Some instructions are cheap (**addition**), others costly (**integer division**). *The cheaper the instructions, the more of them fit in a cycle.*
- ## 3. Execution units are the actual limit
	- A processor contains several **execution units**. Four units capable of addition → up to **4 additions per cycle**. More units, more instructions per cycle. The peak IPC for any given operation is just **how many units can perform it**.
	- This is why capability is **per-operation, not per-chip**, and why vendors differ sharply on the same nominal ISA:
	- | Processor | Multiplications retired per cycle |
	  |---|---|
	  | **x86-64 (Intel / AMD), typical** | **at most 1** — making multiplication relatively expensive vs. addition |
	  | **Recent Apple processors** | **2** |
	  | **AMD Zen 5** | **3** — three execution units capable of multiplication |
	- On execution units alone, a **Zen 5 could theoretically retire 3 additions *and* 3 multiplications in a single cycle**.
- ## 4. And that only counts the scalar registers
	- The numbers above are **conventional multiplications on general-purpose 64-bit registers**.
	- Zen 5 additionally has **four execution units for 512-bit registers, two of which can multiply**. Packing several values into each register means **many multiplications at once** — SIMD width multiplying the scalar width.
	- So "how many multiplications per cycle" has two independent answers (scalar units × vector lanes), and the vector one is much larger.
- ## 5. The road not taken
	- Wider cores are **expensive in transistors**. Those same transistors could have been spent building **more processors** instead.
	- **That is what many people expected** — that computers would come to contain many more general-purpose processors. The industry substantially went the other way: fewer, wider cores.
	- Width is also **much harder than it looks**: "It is not simply a matter of adding execution units. You have to **bring the data to these units**, you have to **order the computation**, **handle the branches**." A design like Zen 5 is called *truly remarkable* for that reason — the scheduling machinery, not the arithmetic, is the achievement.
- ## 6. What it means for programmers
	- **Even when you do not use parallelism explicitly, your code executes in a parallel manner under the hood no matter what.**
	- Practical corollaries worth carrying:
		- **Instruction count is a poor proxy for time.** Two sequences of equal length can differ severalfold if one is multiplication-heavy on a 1-mul/cycle core.
		- **Portable performance intuition doesn't exist at this level.** The same loop has a different ceiling on Apple silicon (2 muls/cycle), Intel (1), and Zen 5 (3).
		- **Dependency chains, not instruction totals, set the floor.** Width is only usable if there is independent work to issue; a strictly serial chain of dependent operations leaves most execution units idle regardless of how wide the core is.
- ## 7. How this fits the wider story
	- This is a **third axis of parallelism**, distinct from the two in [[2026-09-pmpp-ch1-cpu-vs-gpu-design-philosophies]]: not *multi-core* (more complex cores) and not *many-thread* (many simple cores), but **instruction-level parallelism inside one core**, invisible to the programmer.
	- It is the same story from the other side. That note records the **2003 power wall** forcing the multi-core/many-thread fork; this post records what happened to the *individual core* afterward — it kept getting **wider** rather than faster. PMPP's own description of a modern CPU core as "out-of-order, multiple-instruction-issue" is exactly the machinery described here.
	- It also illustrates PMPP's **latency-oriented design** thesis concretely: a wide superscalar core spends an enormous transistor budget to make **one** instruction stream finish sooner — the superlinear price of latency reduction — while the road not taken (more cores) is the throughput-oriented spend.
	- Related: [[2026-09-pmpp-ch1-cpu-vs-gpu-design-philosophies]], [[2026-01-semi-knowledge]], [[2026-05-llm-hw-sw-stack]]
