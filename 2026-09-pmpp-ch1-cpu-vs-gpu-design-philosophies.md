- **Source**: *Programming Massively Parallel Processors*, Chapter 1 "Introduction" (§1.1 Heterogeneous parallel computing, pp. 1–6) and Chapter 5 (§5.2 CUDA memory types, plus the "CPU vs. GPU Register Architecture" sidebar, pp. 98–99). Elsevier, © 2027. DOI [10.1016/B978-0-44-343900-1.00009-3](https://doi.org/10.1016/B978-0-44-343900-1.00009-3). Read from photographed book pages.
- **One-liner**: The 2003 power wall split the microprocessor into two trajectories that never re-merged — **multi-core** kept optimizing the latency of one thread, **many-thread** kept optimizing the throughput of millions — and the resulting ~100× peak-FLOPS gap is not an accident of engineering skill but the direct consequence of where each design spends its chip area and power budget. The same split resurfaces one level down, in the memory hierarchy and the register file.

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

- ## 6. Memory bandwidth — the other half of the gap
	- Graphics performance is **memory-limited, not just arithmetic-limited**: it is bounded by the rate at which data can be delivered from the memory system into the processors and back. A GPU must move extremely large amounts of data in and out of the **graphics frame buffers in its DRAM**, because that movement is what makes video displays rich and satisfying to gamers.
	- **The relaxed memory model is a hardware gift.** Game applications commonly accept a relaxed memory model — a loose contract about how system software, applications, and I/O devices see their memory accesses — which makes it far easier for GPUs to support **massive parallelism in memory access**.
	- **General-purpose processors have no such freedom**: they must satisfy requirements from legacy operating systems, applications, and I/O devices. Those constraints make parallel memory access harder to support, and therefore make **memory bandwidth** harder to increase.
	- Result: graphics chips have been running at roughly **10× the memory bandwidth of contemporaneous CPU chips**, and the book expects GPUs to keep that bandwidth advantage for some time.
	- Worth internalizing: this advantage comes from a **software/compatibility asymmetry**, not from a fabrication or circuit-design advantage. The CPU's bandwidth ceiling is partly the price of backward compatibility.

- ## 7. The cost asymmetry that decides everything
	- The quantitative core of §1.1, and the reason the zero-sum spend in §5 breaks the way it does:
		- **Doubling arithmetic throughput** → double the number of arithmetic units → roughly **2× chip area and 2× power**. Linear.
		- **Halving arithmetic latency** → may require **doubling the current** and **quadrupling the chip area and power**. Superlinear.
	- **Reducing latency is much more expensive than increasing throughput, per unit of area and power.** So GPUs optimize for the **execution throughput of massive numbers of threads** rather than the latency of individual threads — deliberately *allowing* pipelined memory channels and arithmetic operations to have long latency, because that is what makes each unit small.
	- The savings compound: smaller memory-access hardware and smaller arithmetic units mean **more of them fit on a chip**, which raises total execution throughput again.
	- **Fig. 1.1 restated precisely**: (a) CPU = a *smaller* number of *larger* arithmetic units and a *smaller* number of memory channels; (b) GPU = a *larger* number of *smaller* arithmetic units and a *larger* number of memory channels.
	- **Threads are the latency-hiding mechanism.** GPU software is expected to be written with a large number of parallel threads precisely so the hardware can **find other work to do** whenever some threads are waiting on long-latency memory accesses or arithmetic operations.
	- **Why GPU caches are small — and what they are actually for**: not latency reduction, but **bandwidth control**. They exist so that multiple threads accessing the *same* memory data don't all have to go out to DRAM. (Contrast with the CPU's large last-level cache, whose job *is* latency reduction.)
	- Hence the name: **throughput-oriented design** maximizes total execution throughput across many threads *while allowing any individual thread to take a potentially much longer time to execute*.
	- **The honest converse, stated plainly by the book**: GPUs "will not perform well on some tasks on which CPUs are designed to perform well." For programs with **one or very few threads**, a CPU's lower operation latencies deliver **much higher performance** than a GPU. The crossover is thread count, not workload prestige.

- ## 8. Speed is not why a processor wins — three non-performance factors
	- The design of CUDA follows directly from §7's split: since applications have both kinds of parts, **run the sequential parts on the CPU and the numerically intensive parts on the GPU**. NVIDIA introduced the **CUDA programming model in 2007** specifically to support **joint CPU–GPU execution** of one application.
	- Beyond that, the book argues developers pick a processor on factors that can matter **more than speed**:
	- | Factor | The argument | Evidence |
	  |---|---|---|
	  | **Installed base** | Software development cost is only justified by a very large customer population; a processor with small market presence can't attract applications | Traditional parallel computing systems had *negligible* market presence — only a few elite applications, funded by government and large corporations, were ever developed on them. GPUs sold by the **hundreds of millions**; **>1 billion CUDA-enabled GPUs** in use; virtually all desktop PCs and high-end laptops have one |
	  | **Practical form factor / accessibility** | An execution environment that only exists in a data center limits which applications can exist at all | Until 2006 parallel apps ran on data-center servers or departmental clusters. Fine for *publishing a paper* on a 64-node cluster; useless for clinical MRI — **GE and Siemens cannot sell an MRI with racks of compute servers** into a hospital. The **NIH refused to fund parallel programming projects** for some time on exactly this reasoning. Today companies ship MRI products with GPUs, and **NIH funds GPU-computing research** |
	  | **Programmability** | The interface, not the silicon, gated adoption | Until 2006 graphics chips were very hard to use: computation had to be **expressed as a function that paints a pixel**, accessed through **OpenGL or Direct3D**. This was **GPGPU** — General Purpose Programming using a Graphics Processing Unit. Even with higher-level environments, code still had to fit APIs designed to paint pixels, which **limited the kinds of applications** that could be written |
	- The sequencing is the lesson: raw GPU throughput existed *before* 2006, but adoption required removing the pixel-painting abstraction (CUDA, 2007) on top of an installed base that gaming had already paid for. **Hardware capability, market presence, and a usable programming model had to arrive together.**

- ## 9. The CUDA device memory model (Ch. 5, Fig. 5.1)
	- The purpose of having several memory types at all: they exist so the programmer can **improve the compute-to-global-memory-access ratio** of a kernel. By declaring a CUDA variable into one of the memory types, the programmer **dictates its visibility and its access speed** — the type system *is* the performance control.
	- **Fig. 5.1 redrawn** (texture memory omitted, as in the book):
	  ```
	                                 GRID
	  +-------------------------------------------------------------+
	  |  Block (0,0)                      Block (1,0)               |
	  |  +------------------------+       +----------------------+  |
	  |  |     Shared Memory      |       |    Shared Memory     |  |   per-block,
	  |  +------------------------+       +----------------------+  |   on-chip
	  |     ^            ^                   ^            ^         |
	  |     v            v                   v            v         |
	  |  +---------+ +---------+          +---------+ +---------+   |   per-thread,
	  |  |Registers| |Registers|          |Registers| |Registers|   |   on-chip
	  |  +---------+ +---------+          +---------+ +---------+   |
	  |     ^            ^                   ^            ^         |
	  |     v            v                   v            v         |
	  |  +---------+ +---------+          +---------+ +---------+   |
	  |  |Thread   | |Thread   |          |Thread   | |Thread   |   |
	  |  | (0,0)   | | (1,0)   |          | (0,0)   | | (1,0)   |   |
	  |  +---------+ +---------+          +---------+ +---------+   |
	  |       |            |                   |            |       |
	  +-------|------------|-------------------|------------|-------+
	          v            v                   v            v
	  +-------------------------------------------------------------+
	  |   Global Memory     R/W by device,  R/W by host             |   per-grid,
	  +-------------------------------------------------------------+   OFF-chip
	  |   Constant Memory   read-only by device,  R/W by host       |   (DRAM)
	  +-------------------------------------------------------------+
	                              ^
	                              |  host transfers data to/from
	                          +--------+
	                          |  Host  |
	                          +--------+
	  ```
	- **Who may touch what** (the two bullet lists in Fig. 5.1):
		- **Device code can**: R/W **per-thread registers**; R/W **per-thread local memory**; R/W **per-block shared memory**; R/W **per-grid global memory**; **read only** per-grid **constant memory**.
		- **Host code can**: transfer data **to/from per-grid global and constant memories** — and nothing else. The host has no handle on registers, shared, or local memory.
	- | Memory | Scope | Where | Device access | Host access | Character |
	  |---|---|---|---|---|---|
	  | **Registers** | one thread | **on-chip** | R/W | — | Very high speed, highly parallel access; a thread can only touch its own. Kernels use them for frequently accessed per-thread variables |
	  | **Local** | one thread (private) | **off-chip** — it is the thread's own section of global memory | R/W | — | *Same latency as global memory.* Holds what can't live in registers: statically allocated arrays, **spilled registers**, other elements of the thread's **call stack** |
	  | **Shared** | all threads in a block | **on-chip** | R/W | — | High speed but **not as fast as registers**; the efficient means for threads in a block to **cooperate** by sharing input data and intermediate results |
	  | **Global** | whole grid | off-chip DRAM | R/W | R/W | Long latency, relatively low bandwidth |
	  | **Constant** | whole grid | off-chip DRAM | **read-only** | R/W | **Short-latency, high-bandwidth read-only** access from the device |
	- **The trap worth remembering**: "local memory" is local only in *scope*, not in *distance*. It is carved out of global memory, so it carries full global-memory latency — a spilled register is a DRAM access wearing a local variable's name.
	- **Escape hatch**: statically allocated arrays that are accessed **only with constant indices** can still be allocated into registers.
	- (Fig. 5.1 is an incomplete overview — **texture memory** exists but the book does not cover it.)

- ## 10. Why registers matter: the von Neumann boundary (Fig. 5.2)
	- Virtually all modern processors, CUDA devices included, trace back to the **von Neumann model (1945)** — the same model that made the thread a sequential instruction stream back in §2. The global memory of a CUDA device is simply the **Memory box** of that model; the **processor box** is the chip boundary we know today.
	- **The chip boundary**, drawn from the text's description of Fig. 5.2:
	  ```
	  <========= PROCESSOR CHIP =========>||<======== OFF-CHIP ========>
	                                      ||
	     Register File                    ||     Global Memory
	     Shared Memory                    ||     (DRAM technology)
	                                      ||
	     very short access latency        ||     long access latency
	     drastically higher bandwidth     ||     relatively low bandwidth
	                                      ||
	     aggregate register-file bandwidth (all SMs)
	          >= 100x  the global-memory bandwidth
	  ```
	- **Three distinct wins from putting a variable in a register**, which the book is careful to separate:
		- **Latency**: on-chip, so access latency is very short compared with off-chip DRAM.
		- **Bandwidth**: the aggregated bandwidth of all register files across all SMs is **at least two orders of magnitude higher** than global memory's — and, crucially, a register access **no longer consumes off-chip global memory bandwidth at all**. That shows up directly as an **increased compute-to-global-memory-access ratio**.
		- **Instruction count** (the subtler point): an access to a register involves **fewer instructions** than an access to global memory. *(The book's explanation continues past the photographed page — the argument concerns how arithmetic instructions in modern processors encode their operands.)*
	- This is the same §7 economics seen from the software side: the hardware made off-chip access cheap to *build* and expensive to *use*, so the programmer's job is to keep data on-chip.

- ## 11. CPU vs. GPU register architecture (Ch. 5 sidebar)
	- The clearest single instance of the whole note's thesis — the **same component, designed oppositely, because the design objectives differ**.
	- | | **CPU register file** | **GPU register file** |
	  |---|---|---|
	  | **On context switch** | **Saves and restores** the outgoing thread's registers to memory | Nothing is saved — the registers of **all threads scheduled on the processing block stay resident** in the block's register file |
	  | **Switching cost** | Real overhead per switch | **Zero-overhead scheduling** — switching between warps is **instantaneous**, since the incoming threads' registers are already there |
	  | **Size** | Small | **Substantially larger** — it must hold every resident thread's state at once |
	  | **Allocation** | A **fixed** set of registers per thread, regardless of the thread's actual demand | **Dynamic resource partitioning**: an SM may give few registers per thread and run **many** threads, or more registers per thread and run **fewer** |
	  | **Design requirement** | Fixed partitioning | The file itself must be **built to support dynamic partitioning** |
	- **The causal chain**: GPUs hide latency by switching threads (§7) → switching must therefore be free → every resident thread's registers must stay live on-chip → the register file must be huge → and since occupancy is the latency-hiding budget, registers-per-thread must be **tradeable against thread count** at runtime.
	- **Occupancy as the programmer's dial**: registers-per-thread and threads-per-SM trade off directly. Spending more registers per thread buys per-thread speed and costs you the parallelism that was hiding your memory latency in the first place.

- ## 12. Takeaways
	- **Chip history in one line**: frequency scaling (to 2003) → multi-core (parallelism the programmer must expose) → heterogeneous multi-core + many-thread (parallelism at two very different granularities on the same machine).
	- **"Heterogeneous" is the point.** Neither trajectory won. The multi-core CPU remains the right machine for latency-sensitive sequential control flow; the many-thread GPU is the right machine for the computationally intensive, data-parallel parts. Real applications need both, which is why this is a book about heterogeneous parallel *programming*.
	- **The whole GPU design follows from one economic fact**: latency reduction scales superlinearly in area and power, throughput scales linearly. Everything else — small ALUs, long pipelines, tiny caches, thousands of threads, wide memory — is downstream of that.
	- **Two gaps, not one**: ~100× in peak FLOPS *and* ~10× in memory bandwidth. The bandwidth gap comes largely from the relaxed memory model games tolerate and CPUs cannot.
	- **Adoption is a three-legged stool** (installed base, form factor, programming model). Gaming funded the first two; CUDA supplied the third. The AI boom is a tenant in a house built for video games.
	- **The memory hierarchy is the programming model.** CUDA exposes the on-chip/off-chip boundary as *declarations*, so choosing where a variable lives is choosing its speed. Optimization means raising the compute-to-global-memory-access ratio — i.e. moving work inside the chip boundary.
	- **The register file is where both threads of this note meet**: it is enormous on a GPU *because* threads are the latency-hiding mechanism, and dynamically partitioned *because* the number of resident threads is the tuning knob. On a CPU, where a thread is precious and few, a fixed small file and a save/restore context switch are the right answer instead.
	- The 2003 discontinuity is the reason a note like this matters for AI infrastructure at all: the entire modern LLM stack sits on the many-thread branch of a fork that was forced by **heat**, not by an algorithmic insight.
	- Related: [[2026-01-semi-knowledge]], [[2026-06-gpu-connectivity-solutions]], [[2026-05-llm-hw-sw-stack]], [[2026-05-inference-engineering]]
