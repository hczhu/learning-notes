- **Source**: *Programming Massively Parallel Processors*, Chapter 1 "Introduction" (pp. 1–4, §1.1 Heterogeneous parallel computing). Elsevier, © 2027. DOI [10.1016/B978-0-44-343900-1.00009-3](https://doi.org/10.1016/B978-0-44-343900-1.00009-3). Read from photographed book pages.
- **One-liner**: The 2003 power wall split the microprocessor into two trajectories that never re-merged — **multi-core** kept optimizing the latency of one thread, **many-thread** kept optimizing the throughput of millions — and the resulting ~100× peak-FLOPS gap is not an accident of engineering skill but the direct consequence of where each design spends its chip area and power budget.

- ## 1. The virtuous cycle, and the wall it hit
	- Computing has always been **demand-limited, not supply-limited**: applications have consistently wanted more speed and memory than devices could offer. The book's roll-call of insatiable workloads — weather forecast timeliness, accuracy of engineering structural analysis, realism of computer-generated graphics, airline reservations processed per second, fund transfers per second, and (recently) **deep learning**.
	- The **virtuous cycle**: faster hardware → software can provide more functionality and better UI → more useful results → users get accustomed and demand more → industry funds the next generation.
	- **1980s–1990s**: the engine was single-CPU sequential execution — x86 microprocessors from Intel and AMD, driven by **fast increasing clock frequency** plus more hardware resources. Desktops reached **GFLOPS** ($10^9$), datacenters **TFLOPS** ($10^{12}$).
	- **2003 — the wall**: clock-frequency scaling within a single CPU slowed down because of **energy consumption and heat dissipation**. The free lunch of "same binary, faster every generation" ended, and had already been invalid for over a decade by the time this chapter was written.
	- The escape hatch was **more CPUs, not faster CPUs**: virtually all vendors switched to putting **multiple physical CPUs ("processor cores")** on each chip. Crucially this is *not* transparent — a traditional CPU is a single-core CPU, and a multi-core chip **still looks single-core to a user whose work isn't divided into multiple instruction sequences**. This is the moment the burden shifted from hardware to the software developer community; the book calls it the **concurrency revolution**.

- ## 2. Why the switch broke the software model
	- The **von Neumann model** (his seminal 1945 report) is the thing being strained: a program counter holds the memory address of the next instruction, and the processor steps through sequentially.
	- A **thread** = the sequence of instruction-execution activities resulting from that sequential stepwise execution. Formerly a single CPU interleaved multiple instruction sequences; now multiple cores execute them **genuinely in parallel**.
	- Consequence: a **sequential program only ever runs on one of the cores**. Without performance improvement per core, adding features and capabilities to sequential applications reduces growth opportunities for the whole industry.
	- Historical asymmetry worth noting: **parallel programming is not new** — the HPC community had done it for decades — but it ran on large, expensive machines, so only a few elite applications could justify the cost. What changed in 2003 wasn't the technique; it was that **all** microprocessors became parallel, so the addressable set of applications exploded.

- ## 3. The fork: two trajectories since 2003
	- | | **Multi-core** | **Many-thread** |
	  |---|---|---|
	  | **Optimizes for** | Execution speed of *sequential* programs | Execution *throughput* of parallel applications |
	  | **Started at** | Two-core processors, cores growing each semiconductor generation | Large thread counts, growing each generation |
	  | **Exemplars (as of the text)** | Intel/AMD server parts with **over 100 cores**; ARM Ampere with **up to 128 cores** | **NVIDIA Hopper H100**, hundreds of thousands of threads |
	  | **Per-core sophistication** | Out-of-order, multiple-instruction-issue, full x86 ISA, hyper-threading (2 hardware threads/core) | Large number of **simple, in-order** pipelines |
	- The two names describe *what is being scaled*: the multi-core path scales **complex cores**, the many-thread path scales **simple threads**.

- ## 4. The peak-performance gap (2022 numbers)
	- | Precision | H100 GPU peak | Server-grade CPU, same year |
	  |---|---|---|
	  | FP64 (double) | **34 TFLOPS** | |
	  | FP32 (single) | **67 TFLOPS** | *a few* TFLOPS total |
	  | FP16 (half) | **1979 TFLOPS** | |
	- The ratio between many-thread GPU and multi-core CPU peak floating-point throughput has been **increasing for years**, and FP16 shows where the gap really lives — nearly **60× the H100's own FP64 rate**, which is why the deep-learning workload landed where it did.
	- Two honest caveats the book flags immediately:
		- These are **raw speeds, not application speeds**. Chips can potentially support them; delivered application performance is another matter.
		- Peak throughput is not the same as the ability of software to reach it — the rest of the book exists because of that gap.
	- The **"electrical potential" metaphor**: a large gap in peak performance between multi-core and many-thread processors is a charge build-up, and at some point *something has to give*. The book's verdict: **we have reached that point** — the drastically elevated performance of parallel execution has already motivated developers to move the computationally intensive parts of their software to GPUs, and enabled revolutionary new applications (deep learning being the headline). Those computationally intensive parts are, conveniently, **also the prime target of parallel programming**.

- ## 5. Why the gap exists: two design philosophies (Fig. 1.1)
	- The gap is **not** a difference in engineering competence. It is a difference in **what the chip area and power budget are spent on**.
	- | | **CPU — latency-oriented design** | **GPU — throughput-oriented design** |
	  |---|---|---|
	  | **Goal** | Minimize the *effective latency* of arithmetic operations for a single thread | Maximize the *total* floating-point and memory-access throughput |
	  | **ALUs** | Few, large, low-latency — latency minimized at the cost of increased chip area and power **per unit** | Many small ALUs; area and power budget deliberately redirected here |
	  | **Cache** | **Large last-level on-chip caches** — capture frequently accessed data, convert long-latency memory accesses into short-latency cache hits | Small caches (bandwidth control rather than latency hiding) |
	  | **Control** | Sophisticated **branch prediction** and execution control logic to reduce the latency of conditional branches | Minimal control logic per pipeline |
	  | **Memory** | Optimized for latency per access | Optimized for **memory accesses per second** |
	- **The zero-sum argument** (the key insight of §1.1): low-latency arithmetic units, sophisticated operand-delivery logic, large caches, and complex control logic all consume chip area and power **that could otherwise have gone to more arithmetic execution units and more memory access channels**. Latency reduction is *expensive per unit of throughput bought* — CPUs pay it because sequential code has no other way to go faster.
	- **Where the GPU's philosophy came from**: the fast-growing **video game industry**, which exerted tremendous economic pressure for a massive number of floating-point calculations and memory accesses **per video frame**. That demand pushed vendors to maximize the area and power budget dedicated to FP throughput. Notably, the book stresses that the **speed of memory accesses per second is just as, and perhaps even more, important** than FP rate — bandwidth, not just FLOPS, is the throughput-oriented design target.
	- The framing question the chapter opens with — *"why is there such a large peak performance gap?"* — therefore resolves to: **because there is more work to do, i.e. more parallel workers (threads)**, and because the GPU spent its transistors on workers instead of on making one worker's wait shorter.

- ## 6. Takeaways
	- **Chip history in one line**: frequency scaling (to 2003) → multi-core (parallelism the programmer must expose) → heterogeneous multi-core + many-thread (parallelism at two very different granularities on the same machine).
	- **"Heterogeneous" is the point.** Neither trajectory won. The multi-core CPU remains the right machine for latency-sensitive sequential control flow; the many-thread GPU is the right machine for the computationally intensive, data-parallel parts. Real applications need both, which is why this is a book about heterogeneous parallel *programming*.
	- The 2003 discontinuity is the reason a note like this matters for AI infrastructure at all: the entire modern LLM stack sits on the many-thread branch of a fork that was forced by **heat**, not by an algorithmic insight.
	- Related: [[2026-01-semi-knowledge]], [[2026-06-gpu-connectivity-solutions]], [[2026-05-llm-hw-sw-stack]], [[2026-05-inference-engineering]]
