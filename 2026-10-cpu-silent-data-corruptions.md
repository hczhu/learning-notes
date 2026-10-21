file-created-at:: 2026-10-18

- **Source**: Shaobu Wang and Guangyan Zhang, *Revisiting CPU Silent Data Corruptions in Modern Datacenters*, Communications of the ACM, July 31, 2026. Based on the supplied PDF, which synthesizes production incidents and prior studies rather than reporting one new experiment.
- **One-liner**: CPU silent data corruption is a fleet-scale correctness problem with structured, condition-dependent failure patterns; practical protection requires broad testing plus targeted safeguards across hardware, software, and operations.

- ## Key Conclusions

	- **No crash does not mean correct computation.** Silent data corruptions (SDCs) violate data integrity without an immediate, explicit indication. Delayed detection lets bad results propagate, potentially causing data loss, incorrect responses, or failures beyond recovery capability.
	- **Internal processor faults matter, not just radiation or noise.** Design bugs and manufacturing defects can escape factory testing; transistor weakness and wear emerge during service. Timing marginalities can make a circuit miss its timing requirements under particular voltage or temperature conditions.
	- **Rare per processor can be significant across a fleet.** Google, Meta, and Alibaba report low but meaningful faulty-processor prevalence. Larger deployments expose more faults even without an increase in the underlying per-processor prevalence.
	- **Detection without correct diagnosis is insufficient.** In one storage incident, corrupted CPU checksum calculations were attributed to disk faults. Error handling consumed resources while the faulty CPU kept generating errors. Software or library updates can also expose dormant faults by exercising different instructions.

- ## Key Data Points

	- Figures below are reported in the CACM article from different studies, not one pooled dataset. Reference links point to the article's bibliography.
	- | Measure | Reported data point | Source / scope | How to interpret it |
	  | --- | --- | --- | --- |
	  | SDC-related processor-fault prevalence | A few occurrences per several thousand processors | Google and Meta; [refs. 14](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B14), [20](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B20) | Approximate fleet observation; the article does not give an exact percentage for either vendor. |
	  | SDC-related processor-fault prevalence | **3.61‱ = 3.61 per 10,000 = 0.0361% = 361 per million** | Alibaba Cloud; measured across **over one million production processors**; [ref. 38](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B38) | The per-million figure is a unit conversion, not a reported incident count. This is not an annualized rate or a per-instruction error probability. |
	  | Traditional external-source SDC expectation | Approximately **one event per 1,000 years in a production datacenter** | Historical assumption cited in [ref. 31](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B31) | A different fault model and denominator, not a directly comparable estimate of internally faulty processors. |
	  | Magnitude of float64 errors | **97.5%** of observed relative precision losses were **below 0.01%** | Alibaba Cloud measurements; Figure 2 and [ref. 39](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B39) | Many corruptions are numerically subtle and can evade broad result-range checks. |
	  | Very small numerical corruption | **0.0000003%** relative precision loss | One Google case; [ref. 35](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B35) | Illustrative case, not a typical error magnitude or fleet-wide distribution. |
	  | Multiple-bit corruption | Approximately **20%** of cases had **two** flipped bits; approximately **8%** had **more than two** | Extended-double-precision observations; [ref. 38](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B38) | Multi-bit corruption is substantial in these observations; do not generalize the percentages to all CPU operations. |
	  | Error occurrence frequency | **0.01 errors/minute to several hundred/minute** | Studied faulty processors under triggering conditions; [ref. 39](https://cacm.acm.org/research/revisiting-cpu-silent-data-corruptions-in-modern-datacenters/#B39) | Measures reproducibility once a fault is exercised, not how many processors in the fleet are faulty. |
	- **Scale insight**: CPU SDCs remain relatively infrequent compared with other datacenter errors, but million-processor deployments make them operationally important. More observed incidents need not imply that individual processors have become less reliable.

- ## Production Evidence Changes the Fault Model

	- **Bit flips are not uniformly distributed.** Observed faults favor particular components and bit positions, including floating-point fractional bits. One fault can repeatedly affect the same positions, and some require specific input values or bit patterns.
	- **Small numerical errors can evade detection.** Some iterative methods tolerate the small perturbations observed in the data table, but this does not imply all workloads are safe; broad expected-value checks can miss them.
	- **Multiple-bit corruption is not negligible.** Single-bit fault models miss important observed behaviors.
	- **A persistent defect can produce intermittent errors.** Low reproducibility complicates diagnosis; high reproducibility makes rapid isolation important.
	- **Operating conditions are part of the test.** Temperature, voltage, frequency, and workload affect triggering. A Google case worsened at a lower frequency because voltage also fell; an Alibaba test update reduced coverage because it ran cooler. Lower frequency or a newer test suite is not automatically safer.

- ## Why No Single Defense Suffices

	- | Defense | Main value | Main limitation |
	  | --- | --- | --- |
	  | Periodic/online processor testing | Broad coverage without application changes; helps reproduce and diagnose faults. | Exhaustive corner-case testing is costly, and aging introduces new faults. |
	  | Operating-condition control | Avoids known triggering conditions. | Can cost performance; complements testing rather than repairing existing corruption. |
	  | Monitoring indirect symptoms | Low-overhead clues, such as core dumps. | Indirect signals cannot reliably detect every silent error. |
	  | Redundant computation | Compare independent executions to detect disagreement; additional replicas can support correction. | High compute/resource cost. |
	  | ECC, checksums, erasure coding | Protect stored or transferred data within their coverage. | Not general protection for computation; some codes cannot correct observed multi-bit faults, and checksum/parity computation can itself be corrupted. |
	  | Predicted result ranges | Detects implausible outputs in suitable numerical workloads. | Small errors can remain within the range; depends on workload properties. |
	  | Algorithm-based fault tolerance (ABFT) | Maintains mathematical relationships alongside computation and checks them afterward, often at low overhead. | Each scheme covers specific algorithms, not arbitrary application logic. |

- ## Practical Implications and Research Directions

	- **Layer defenses selectively.** Use fleet testing as a baseline, then add redundancy or ABFT to operations that both exercise vulnerable processor features and require strong correctness. Protecting only storage/transfer data paths leaves other computations exposed.
	- **Manage triggering conditions deliberately.** Record and control temperature, voltage, frequency, and relevant inputs during diagnosis and testing. A passing test outside the fault's triggering conditions is weak evidence of health.
	- **Target vulnerable components, not just large ones.** Architectural complexity can make a relatively small floating-point unit vulnerable. Selective hardware redundancy and component-specific operating controls may improve protection per unit cost.
	- **Extend the work beyond CPUs, without assuming identical behavior.** GPUs, TPUs, and DPUs also face SDC risks, but their workloads and fault implications need separate characterization. Large-scale industry data is essential because individual incidents are rare.
	- **Bottom line**: no evaluated strategy combines universal coverage, strong effectiveness, and low overhead. The authors favor coordinated protection across layers, accepting narrower coverage for expensive safeguards to make deployment economical.
