- **Source**: Meta Engineering, "Meta's Generative Ads Model (GEM): The Central Brain Accelerating Ads Recommendation AI Innovation," by Huayu Li, Xiaoyi Liu, Jade Nie, Ellie Wen, Chunzhi Yang, Jiyan Yang, Nancy Yu, Habiya Beg, Gil Arditi, and Neeraj Bhatia, posted November 10, 2025. Source provided as local PDF: `Meta’s Generative Ads Model (GEM)_ The Central Brain Accelerating Ads Recommendation AI Innovation - Engineering at Meta.pdf`.
- **One-liner**: GEM is Meta's LLM-scale foundation model for ads recommendation: a large, multi-domain, multimodal recommender trained on thousands of GPUs, then post-trained / distilled into many latency-sensitive vertical ads models.

- ## Key Data Points
	- | Metric / claim | Data point | Why it matters |
	  |---|---:|---|
	  | Instagram ad conversions after GEM launch | +5% in Q2 | Direct production business impact from GEM transfer into ads serving |
	  | Facebook Feed ad conversions after GEM launch | +3% in Q2 | Confirms effect across more than one major surface |
	  | Q3 architecture improvement | 2x performance benefit per given data / compute | Scaling curve improved, not just absolute compute increased |
	  | Architecture vs original ads ranking models | 4x more efficient at driving ad-performance gains for given data / compute | GEM improves scaling economics versus prior rankers |
	  | Knowledge transfer framework | 2x effectiveness of standard knowledge distillation | The foundation model's value depends on transferring knowledge into production VMs |
	  | Training stack | 23x increase in effective training FLOPs | Training infra was rebuilt, not simply scaled linearly |
	  | GPU scale | 16x more GPUs | Demonstrates LLM-like training scale for RecSys |
	  | Model FLOPS utilization (MFU) | 1.43x increase | More GPUs were used with higher efficiency, avoiding pure brute force |
	  | Job startup time | 5x reduction | Improves experiment velocity and reduces idle GPU time |
	  | PyTorch 2.0 compile time | 7x reduction via caching | Compilation overhead became material at this scale |
	  | Long sequence length | Up to thousands of events | GEM can model much longer user journeys than compact-vector approaches |
	  | Experiment support | Lightweight variants support over half of all experiments | Full-size GEM is too expensive for every iteration; small variants are part of the workflow |

- ## What GEM Is
	- GEM is Meta's ads recommendation foundation model, built with an LLM-inspired scaling paradigm but targeted at RecSys rather than text generation.
	- Meta describes it as the industry's largest foundation model for recommendation systems, trained at large-language-model scale.
	- GEM is not just a single serving model. It functions as a central foundation model whose knowledge is propagated to many downstream ads models.
	- The production objective is joint optimization across the ads funnel:
		- awareness
		- engagement
		- conversion
		- user relevance / preference alignment
		- advertiser ROI / ROAS
	- The key architectural bet: a huge ads foundation model can learn broad cross-surface, cross-domain representations that smaller vertical models cannot learn alone.

- ## Core RecSys Challenges GEM Addresses
	- **Large dynamic feature space**:
		- Meta sees billions of user-ad interactions daily.
		- High-value labels like clicks and conversions are sparse.
		- The model must generalize across users, ads, behaviors, surfaces, and objectives despite strong class imbalance.
	- **Heterogeneous data**:
		- Inputs include advertiser goals, ad creatives, formats, measurement signals, user behaviors, and delivery channels.
		- GEM must unify ads and organic interactions across multiple Meta apps.
	- **Training scale**:
		- Model size and embedding scale require thousands of GPUs.
		- Sparse embedding components and dense neural components require different parallelism strategies.
	- **Production transfer**:
		- GEM itself is too large / general to directly replace every latency-sensitive ads model.
		- Its knowledge must be compressed into hundreds of user-facing vertical models.

- ## Input Feature Design
	- Meta groups GEM features into two broad categories:
		- **Sequence features**: activity history, ad / content clicks, views, interactions, and long user behavior traces.
		- **Non-sequence features**: user and ad attributes such as age, location, ad format, creative representation, and other contextual attributes.
	- GEM applies customized attention mechanisms to each feature group independently while also enabling cross-feature learning.
	- The design goal is to scale both:
		- **Depth**: more layers / deeper interaction modeling.
		- **Breadth**: more feature types / broader surface coverage.

- ## Architecture: Non-Sequence Feature Interaction
	- GEM extends the Wukong architecture for non-sequence feature interactions.
	- The core technique is stackable factorization machines with cross-layer attention connections.
	- Purpose:
		- capture which user-ad feature combinations matter most
		- learn complex high-order interactions
		- support deeper and broader feature interaction modeling
	- Scaling axes:
		- **Vertical scaling**: deeper interaction modeling across layers.
		- **Horizontal scaling**: broader feature coverage across many user / ad / context features.
	- Technical insight: ads recommendation is not just sequence modeling; dense cross-features like user attributes × ad creative × placement still carry large signal.

- ## Architecture: Offline Sequence Feature Modeling
	- GEM models long user behavior sequences spanning ads and organic interactions.
	- Traditional compressed-vector sequence summaries can discard important behavioral information.
	- GEM uses a pyramid-parallel structure:
		- multiple parallel interaction modules
		- stacked in a pyramid-like formation
		- designed to capture complex user-ad relationships at scale
	- Meta's new offline feature infrastructure can process sequences of up to thousands of events with minimal storage cost.
	- Why this matters:
		- Ads conversion paths can be long and sparse.
		- A user's purchase journey may include many organic content interactions before an ad click or conversion.
		- Longer history improves intent inference and cross-surface personalization.

- ## Architecture: Cross-Feature Learning With InterFormer
	- GEM uses an InterFormer-style design for cross-feature learning.
	- Instead of first compressing sequences into a small vector and then passing that vector downstream, GEM preserves more of the full sequence information.
	- Design pattern:
		- parallel summarization
		- interleaving structure
		- alternating sequence-learning layers and cross-feature interaction layers
	- Sequence-learning layers can use custom transformer architectures.
	- Cross-feature layers allow interaction between sequence-derived information and non-sequence attributes.
	- Technical insight:
		- Large RecSys models need sequence modeling and tabular / categorical feature interaction at the same time.
		- A pure transformer over all raw features would be too expensive; a pure tabular model would lose journey structure.

- ## Multi-Domain Learning
	- GEM learns across Meta surfaces such as:
		- Facebook
		- Instagram
		- Business Messaging
	- Different surfaces have different behavior distributions and optimization targets.
	- GEM tries to balance:
		- **shared cross-surface learning**: e.g. Instagram video ad engagement can inform Facebook Feed predictions.
		- **domain-specific optimization**: each surface still needs predictions tailored to its objective, such as clicks or conversions.
	- Technical insight: foundation RecSys models need multi-task / multi-domain learning, not one identical ranking function for every surface.

- ## Post-Training And Knowledge Transfer
	- GEM's production value depends on transferring knowledge into hundreds of smaller vertical models (VMs).
	- Meta uses two transfer paths:
		- **Direct transfer**:
			- GEM transfers knowledge to major VMs in the same data spaces where GEM was trained.
		- **Hierarchical transfer**:
			- GEM teaches domain-specific foundation models.
			- Domain-specific FMs then teach VMs.
	- Techniques used:
		- knowledge distillation
		- representation learning
		- parameter sharing
	- The combined transfer framework is reported to be 2x as effective as standard knowledge distillation.
	- Important production principle: the large FM does not have to serve every request directly if it can cheaply improve the smaller serving models.

- ## Knowledge Distillation Details
	- Problem: VMs can receive stale or misaligned supervision.
	- Sources of staleness / mismatch:
		- delays in FM training
		- delays in FM evaluation
		- domain mismatch between GEM predictions and surface-specific VM objectives
	- Meta uses a **Student Adapter** during training.
	- Student Adapter role:
		- lightweight transformation module
		- refines teacher outputs using recent ground-truth data
		- aligns GEM / FM predictions with observed outcomes
		- gives student VMs more current and domain-relevant supervision
	- Technical insight: at Meta scale, distillation is not just teacher logits → student logits; the teacher signal itself needs calibration against fresh labels.

- ## Representation Learning
	- GEM generates compact, semantically meaningful features from raw data.
	- These representations support downstream tasks like ad click prediction.
	- Representation learning complements distillation by transferring feature knowledge even when full model behavior cannot be copied.
	- Meta emphasizes that this can improve FM-to-VM transfer without adding inference overhead.
	- Technical insight: representation transfer is attractive in ads because serving latency budgets are tight and full teacher inference may be too expensive.

- ## Parameter Sharing
	- Parameter sharing lets smaller VMs selectively reuse components from foundation models.
	- Goal:
		- reduce redundancy
		- reuse rich pre-learned patterns
		- avoid full FM compute cost at inference time
	- This is especially important for latency-sensitive VMs.
	- Technical insight: Meta is exploring a spectrum between full model distillation and direct component reuse; VMs do not have to be fully independent from the foundation model.

- ## Training Stack
	- GEM required a re-engineered training stack because its scale is closer to LLM training than traditional ads ranker training.
	- Main scaling techniques:
		- multi-dimensional parallelism
		- custom GPU kernels
		- model-system co-design
		- near-linear scaling across thousands of GPUs
	- Dense components:
		- use Hybrid Sharded Distributed Parallel (HSDP)
		- optimize memory usage and communication cost for dense parameters
	- Sparse components:
		- large embedding tables for user / item features
		- use a two-dimensional strategy combining data parallelism and model parallelism
		- optimized for synchronization efficiency and memory locality
	- Technical insight: RecSys foundation models are harder than pure dense LLMs in one way because dense neural layers and huge sparse embeddings require different distributed-training strategies in one job.

- ## GPU Throughput Optimizations
	- Meta implemented custom in-house GPU kernels for:
		- variable-length / jagged user sequences
		- computation fusion
		- newer GPU hardware features
	- Meta uses graph-level compilation in PyTorch 2.0 for:
		- activation checkpointing
		- operator fusion
		- execution efficiency
	- Memory optimizations:
		- FP8 quantization for activations
		- unified embedding formats
		- reduced memory footprint
	- Communication optimization:
		- Meta developed GPU communication collectives using NCCLX, Meta's fork of NVIDIA NCCL.
		- These collectives can operate without consuming Streaming Multiprocessor (SM) resources.
		- Purpose: avoid contention between communication and compute, improving overlap and utilization.
	- Technical insight: at this scale, GPU efficiency is not just model architecture; startup time, compile time, communication overlap, jagged sequence kernels, and embedding formats all materially affect the feasible model frontier.

- ## Training Agility And Lifecycle Efficiency
	- Meta optimizes **effective training time (ETT)**:
		- the proportion of training time spent processing new data
		- lower overhead means less GPU idleness
	- Startup overhead reductions:
		- trainer initialization
		- data reader setup
		- checkpointing
		- PyTorch 2.0 compilation
	- Lightweight model variants:
		- used in exploration
		- much cheaper than full GEM
		- support over half of all experiments
	- Post-training workload:
		- GEM runs forward passes to generate labels and embeddings for downstream models.
		- Unlike many LLM deployment setups, Meta performs continuous online training to refresh foundation models.
	- Traffic / compute sharing:
		- between training and post-training knowledge generation
		- between foundation model and downstream models
	- Technical insight: the development lifecycle is part of model scaling. If every idea requires a full-size run, the system cannot improve quickly enough.

- ## Future Direction
	- GEM will expand to learn from the full Meta ecosystem:
		- organic interactions
		- ads interactions
		- text
		- images
		- audio
		- video
	- Meta's stated direction is a unified engagement model that can rank both organic content and ads.
	- Future model capabilities:
		- larger clusters
		- newer AI hardware
		- stronger multimodal understanding
		- inference-time scaling
		- intent-centric user journeys
		- agentic advertiser automation
	- Technical insight: Meta is moving from isolated ads rankers toward a central multimodal model that understands user intent across organic and paid surfaces.

- ## Mental Model
	- GEM is to Meta ads what an LLM foundation model is to NLP applications:
		- train a very large general model
		- extract broad representations
		- transfer knowledge into specialized smaller models
		- use post-training to adapt to production tasks
	- But GEM differs from LLMs in key ways:
		- sparse embeddings are central
		- labels are delayed and sparse
		- serving models are latency constrained
		- continuous online refresh matters
		- multi-domain objectives are explicit
		- full-model inference is often not the production endpoint

- ## Open Questions
	- How large is GEM in parameters, embedding-table size, training tokens / examples, and GPU-hours? The article gives relative scale metrics but not absolute model size.
	- How much of GEM's conversion lift comes from better ranking versus better retrieval, budget pacing, auction dynamics, or creative selection?
	- What is the trade-off between cross-surface learning and domain-specific overfitting?
	- How often is GEM refreshed, and what is the freshness gap between online labels and downstream VM updates?
	- What share of GEM's value comes from long sequence modeling versus non-sequence cross-feature interaction?
	- Can the same GEM-style foundation model rank organic content and ads without creating objective conflicts between user engagement and advertiser value?
	- How does Meta control privacy, consent, and data governance when a single model learns across ads, organic interactions, and multiple modalities?
