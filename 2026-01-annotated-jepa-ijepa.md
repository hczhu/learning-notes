- **Source**: Elon Litman, "The Annotated JEPA" (filed under *Deep Learning*), 27 Jan 2026. A from-scratch, annotated walkthrough of Joint Embedding Predictive Architectures — "doing for JEPA what *The Annotated Transformer* did for the Transformer." Builds I-JEPA in PyTorch, then extends to V-JEPA / V-JEPA 2 / LeJEPA.
- **One-liner**: JEPA is Yann LeCun's answer to self-supervised learning without labels — **predict in representation (latent) space, not pixel space** — so the model learns semantic structure and ignores high-entropy noise, with an anti-collapse mechanism (EMA teacher, or a distributional regularizer) doing the load-bearing work.

- ## The Core Idea
	- **The problem**: self-supervised representation learning needs an objective that captures meaningful structure without labels — and without collapsing to trivial solutions or wasting capacity on irrelevant detail.
	- **JEPA's answer**: train by prediction, but **predict in representation space**. See a context $x$, encode a hidden target region $y$ to $s_y$, and have a predictor guess $\hat{s}_y$ from the context. Minimize distance $D(\hat{s}_y, s_y)$.
	- **Why it forces good features** (the "forcing function"):
		- The context encoder can only succeed if its encoding $s_x$ contains enough information to determine $s_y$ — i.e. it must extract **semantic, structural features** (object identity, spatial relations, physical constraints). Pixel-level noise in $x$ doesn't help predict $s_y$, so the encoder learns to discard it.
		- The **target encoder has complementary pressure**: its output $s_y$ must be *predictable* from context. Random high-frequency texture couldn't be predicted, so the target encoder learns to output representations capturing the **shared structure** between $x$ and $y$, not idiosyncratic details.

- ## Prediction vs. Reconstruction
	- Two semantically identical patches can differ in hundreds of pixels (lighting, texture, JPEG artifacts). A **reconstruction** objective must explain all that noise — wasting capacity on high-entropy detail.
	- JEPA's loss lives in **embedding space**, so the target encoder can discard nuisance detail and the model captures only what the encoder preserves. "The architecture resembles generative models, but the loss lives in embedding space, not input space."

- ## The JEPA Template (LeCun's general formulation)
	- Starts from paired, semantically related views $(x, y, z)$: $x$ = observation, $y$ = what to predict, $z$ = optional latent for unknown factors (makes prediction multimodal). Requirement: knowing $x$ should constrain what $y$ can be.
	- **Three parts**:
		- Encode both views into a shared space: $s_x = f_\theta(x)$, $\quad s_y = f_{\bar\theta}(y)$
		- Predict target from context: $\hat{s}_y = g_\phi(s_x, z)$
		- Minimize prediction error in representation space: $\mathcal{L}(\theta, \bar\theta, \phi) = D(\hat{s}_y, s_y)$
	- Encoders $f_\theta$, $f_{\bar\theta}$ may share weights or not — weight sharing trades inductive bias against flexibility; different modalities force separate encoders.
	- The **latent $z$** handles genuine uncertainty (e.g. a car at an intersection could turn left or right). Without $z$, a deterministic predictor is forced to hedge and matches neither future. With $z$, the predictor learns a *family* of predictions; energy is minimized over $z$: $E(x,y) = \min_z E(x,y,z)$. Masked image modeling usually doesn't need $z$ (context determines target up to noise); long-horizon/video prediction does.

- ## Collapse — The Central Failure Mode
	- The easiest way to minimize a matching loss: **output the same constant vector for everything** → loss = 0, representations useless. Every joint-embedding method must prevent this.
	- Anti-collapse strategies (menu): contrastive negatives (expensive), explicit covariance constraints (tricky to tune), stop-gradient (surprisingly effective), or **teacher–student encoders with EMA updates (I-JEPA's choice)**.
	- **Why EMA prevents collapse**: if both encoders trained by gradient descent, the target encoder would drift toward easy-to-predict (constant) outputs and the context encoder would follow → mutual collapse. Instead, the target (teacher) updates as a lagging exponential moving average of the context (student):
	  $$
	  \theta_{\text{target}} \leftarrow m\,\theta_{\text{target}} + (1-m)\,\theta_{\text{context}}, \quad m \approx 1
	  $$
	  Targets are stable on the timescale of a gradient step, so the context encoder can't exploit fast-moving targets to find degenerate solutions — it must actually learn to predict the slowly-evolving teacher's outputs.

- ## I-JEPA (the image instantiation)
	- Both $x$ and $y$ come from the **same image**: $x$ = one large **context block** of patches; $y$ = several smaller **target blocks** elsewhere. Predict target-block representations from the context block.
	- **Three key design choices**:
		- **Patch tokens** (ViT-style): a 224×224 image with 16×16 patches → 196 tokens in a 14×14 grid.
		- **Masking strategy**: predict target blocks that are *sufficiently large in scale*, using a context block that is *informative and spatially distributed*. Typical sampling: $M = 4$ target blocks, aspect ratio range (0.75, 1.5), scale range (0.15, 0.2); context block scale range (0.85, 1.0), unit aspect ratio; **any target/context overlap is removed** to keep the task non-trivial.
		- **Mask the target encoder's *output*, not its input**: the target encoder sees the *full* image and produces high-level representations; then you select which to predict. Crucial for high-semantic-level targets.
	- These choices *replace hand-designed multi-view augmentations* with a structured prediction task — that's I-JEPA's inductive bias.
	- **Three networks**:
		- **Context encoder (student)**: a ViT processing only visible context patches → context embeddings. Efficiency comes from processing only visible patches.
		- **Target encoder (teacher)**: a ViT processing the full image → embeddings for all patches; updated by **EMA** of context-encoder weights, not gradients.
		- **Predictor**: a smaller Transformer taking context embeddings + mask tokens (a shared learnable vector + positional embedding) for target positions → predicted target embeddings.
	- **Loss**: MSE between predicted and target-encoder patch representations, i.e. $D(\hat{s}_y, s_y) = \lVert \hat{s}_y - s_y \rVert_2^2$. Non-generative: **no pixel decoder**, loss on embeddings only.

- ## V-JEPA & V-JEPA 2 (extending to video)
	- Video amplifies the case for latent-space prediction: frames are redundant, pixels are noisy, and the dynamics that matter live in abstraction. (Precursor: **MC-JEPA**, July 2023 — disentangled *motion* vs. *content* with a shared encoder.)
	- **V-JEPA**: ViT over a **3D grid of spatiotemporal patches**; masks **3D spatiotemporal blocks** (vs. 2D in I-JEPA), forcing reasoning about temporal coherence and motion. Same EMA-teacher framework, loss in representation space. Learns strong video reps **without** temporal augmentations, optical flow, or pixel reconstruction; transfers to action recognition.
	- **V-JEPA 2**: same objective, but the model supports three modes:
		- **Understanding**: pretrained encoder → features for classification / action recognition.
		- **Prediction**: forecast future *latent* states from a video prefix (forecasting abstract world state, not pixels).
		- **Planning (V-JEPA 2-AC)**: freeze the encoder, fine-tune the predictor to take **robot actions & poses as the latent $z$** → a learned **dynamics model in latent space**. An agent plans by search: roll out candidate action sequences via the predictor, pick the trajectory closest to the goal representation. Tractable precisely because the representation has compressed away irrelevant detail.

- ## LeJEPA (a JEPA without heuristics)
	- Paper: Balestriero & LeCun, "LeJEPA: Provable and Scalable Self-Supervised Learning Without the Heuristics," arXiv 2511.08544 (Nov 2025). Critiques exactly the I-JEPA-style engineering: advertises **no stop-gradient, no teacher–student, no hyperparameter schedulers**; linear time/memory; single trade-off hyperparameter; ~50 lines of code.
	- **Core idea**: treat "avoiding collapse" as a **distribution-matching problem**. Argues that to minimize downstream prediction risk, JEPA embeddings should follow an **isotropic Gaussian** (maximally, symmetrically spread — the opposite of a collapsed, concentrated distribution).
	- **SIGReg (Sketched Isotropic Gaussian Regularization)**: sample random directions $w$, project embeddings onto each ($w^\top s_i$); if truly isotropic Gaussian, these 1-D projections are univariate Gaussian, so penalize deviation from Gaussianity (via characteristic-function matching), averaged over many directions.
		- Justification: **Cramér–Wold theorem** — a distribution is uniquely determined by its 1-D projections. Sampling random projections instead of the full $D \times D$ covariance makes SIGReg scale **linearly** in dimension and batch size.
	- **Objective**: $\mathcal{L}_{\text{LeJEPA}} = \mathcal{L}_{\text{predict}} + \lambda\,\mathcal{L}_{\text{SIGReg}}$ ("Latent Euclidean JEPA"). Reported: ViT-H/14 reaches **79%** on ImageNet with frozen backbone; validated on 10+ datasets, 60+ architectures.
	- **Contrast with I-JEPA**: I-JEPA's stability comes from *architectural asymmetry* (EMA teacher + mask strategy); LeJEPA's comes from *regularizing the embedding distribution*, claimed stable across architectures/domains/hyperparameters.

- ## JEPA as an Energy-Based Model & Path to World Models
	- LeCun frames JEPA as an **energy-based model**: $E(x,y) = D\big(g_\phi(f_\theta(x)), f_{\bar\theta}(y)\big)$ — energy is prediction error in representation space; low for semantically related $(x,y)$.
	- Unlike contrastive methods, JEPA doesn't train on negative pairs; anti-collapse mechanisms (EMA, distributional regularizers) keep the energy surface informative instead. EBMs avoid intractable partition functions and admit flexible inference (find $y$ minimizing energy) — exactly what planning needs.
	- **The template generalizes** beyond vision: A-JEPA (audio), Graph-JEPA (graphs), GeneJEPA (single-cell genomics). Whenever data has a natural context/target notion, you can instantiate JEPA. Design becomes engineering questions: what are context/target? what representation space? what predictor? what anti-collapse mechanism?
	- **LeCun's world-model vision**: JEPA is the predictive core of a modular autonomous-intelligence architecture — *configurator* (executive control), *perception*, *world model* (JEPA), *cost module* (intrinsic cost + trainable critic), *actor*, *short-term memory*. A representation-space world model makes planning tractable: search abstract state trajectories instead of hallucinating pixel-level futures.
	- **Argument against pure autoregressive LLMs**: LLMs predict next tokens in a *discrete, compressed* symbol space; they can describe how a bicycle works without any internal model of balance or momentum. JEPAs operate on raw sensory data, so predicting well *forces* learning physical-world structure (objects persist, physics constrains motion, causes precede effects) — an inductive bias LLMs may lack.

- ## Bottom Line
	- I-JEPA, V-JEPA, and LeJEPA are existence proofs that **prediction in a learned abstract space** works, scales to video, and can ground planning. JEPA is a coherent alternative to the three dominant paradigms — generative pixel modeling, contrastive learning with hand-designed augmentations, and autoregressive token prediction — where **the representation itself determines what is worth modeling**.
	- Related: [[2023-02-toolformer-lms-can-teach-themselves-to-use-tools]], [[2026-07-is-ai-intelligent-meyer]]
