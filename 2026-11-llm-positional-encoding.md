tags:: [[LLM]], [[positional-encoding]], [[RoPE]], [[ai-model:GPT-6 Astra]], [[ai-effort:high]]
file-created-at:: 2026-11-18

- # LLM positional encoding
- **Source**: User-provided screenshot, Figure 14, “MiniMax-M3 applies RoPE to queries and keys inside attention.” Diagram credited to Sebastian Raschka in the screenshot; original publication URL/date not supplied. Model specifications below reproduce the screenshot, rather than independently verified specifications.
- **Key observation**: In this architecture, positional information enters through **rotary position embedding (RoPE) on queries and keys inside attention**, not through an extra positional vector added at the token-embedding layer.

- ## Architecture: where RoPE enters
	- Read **bottom to top**, matching the screenshot. The two `+` nodes are residual additions. The repeated block represents 60 sequential blocks, not a loop back into the same block.
	- Q/K and V projection operations are implicit in the original figure; the edge labels below make those paths explicit without drawing separate projection layers. QK-Norm and RoPE apply to the Q/K path, not the V path.
	- ```mermaid
	  flowchart BT
	      input["Input text"] --> tokens["Tokenized text"]
	      subgraph model["MiniMax-M3 — architecture shown in screenshot"]
	          direction BT
	          embed["Token embedding layer<br/>Embedding dimension: 6,144"]
	          subgraph block["Transformer block × 60"]
	              direction BT
	              x["Block input"] --> norm1["RMSNorm 1"]
	              norm1 -->|"Q and K projections — implicit"| qknorm["QK-Norm"]
	              qknorm --> rope["RoPE<br/>Positional information enters here"]
	              rope --> attn["GQA / MiniMax Sparse Attention"]
	              norm1 -->|"V projection — implicit"| attn
	              attn --> add1(("+"))
	              x -->|"Residual connection"| add1
	              add1 --> norm2["RMSNorm 2"]
	              norm2 --> ffn["MoE<br/>Dense FFN in first 3 blocks"]
	              ffn --> add2(("+"))
	              add1 -->|"Residual connection"| add2
	          end
	          embed --> x
	          add2 --> finalnorm["Final RMSNorm"]
	          finalnorm --> output["Linear output layer<br/>Vocabulary size: 200k"]
	      end
	      tokens --> embed
	      output --> logits["Output logits"]
	      context["Screenshot annotation:<br/>supported context length: 1M tokens"] -.-> rope
	      exception["First 3 blocks:<br/>GQA + dense FFN<br/>FFN hidden size: 12,288<br/>instead of sparse attention + MoE"] -.-> ffn
	      classDef positional fill:#6d28d9,stroke:#4c1d95,color:#fff,stroke-width:3px;
	      class rope positional;
	  ```
	- **Rendering**: The editable source is a Mermaid fenced code block. GitHub renders Mermaid; Logseq needs a compatible renderer, such as [Diagram Blocks](https://github.com/nanodeath/logseq-diagram-blocks/), to display fenced Mermaid blocks. Without a renderer, the diagram remains readable/editable as source. No plugin is installed by this note.

- ## RoPE close-up: from one token to an attention score
	- **Source**: Three additional user-provided screenshots, Figures 15–17, credited to Sebastian Raschka: projection and head reshaping, pairwise rotations, and the first split-half pair. This is a **toy example** with 16 hidden features and two 8-feature heads, not the MiniMax-M3 dimensions above.
	- Read **top to bottom**. Position `t` is a placeholder: use `t = m` for the query token and `t = n` for the key token. At any one position, both projections receive the same hidden state. The shared pairing/rotation boxes describe a rule applied **independently** to each Q/K head; they do not merge queries and keys.
	- ```mermaid
	  flowchart TD
	      token["Token at position t<br/>Example: fox at t = m"] --> lookup["Embedding lookup"]
	      lookup --> embedding["Token embedding: 16 features"]
	      embedding --> earlier["Earlier transformer blocks, if any<br/>Typically preserve hidden-state width"]
	      earlier --> hidden["Hidden state h_t: 16 features<br/>Same input to Q and K projections"]
	      hidden --> wq["Learned query weights W_Q<br/>16 × 16"]
	      hidden --> wk["Learned key weights W_K<br/>16 × 16"]
	      wq --> q["Query vector q_t<br/>Shape: 16"]
	      wk --> k["Key vector k_t<br/>Shape: 16"]
	      q --> qheads["Reshape to 2 × 8<br/>Q head 0: slots 1–8<br/>Q head 1: slots 9–16"]
	      k --> kheads["Reshape to 2 × 8<br/>K head 0: slots 1–8<br/>K head 1: slots 9–16"]
	      qheads -->|"Each head independently"| halves
	      kheads -->|"Each head independently"| halves
	      subgraph pairing["Split-half pairing — zoom into head 0"]
	          halves["8-feature head<br/>First half: a0, a1, a2, a3 — slots 1–4<br/>Second half: b0, b1, b2, b3 — slots 5–8"]
	          halves --> p0["Pair 0: (a0, b0)<br/>Slots (1, 5) — one from each half"]
	          halves --> p1["Pair 1: (a1, b1)<br/>Slots (2, 6)"]
	          halves --> p2["Pair 2: (a2, b2)<br/>Slots (3, 7)"]
	          halves --> p3["Pair 3: (a3, b3)<br/>Slots (4, 8)"]
	      end
	      p0 --> rotate
	      p1 --> rotate
	      p2 --> rotate
	      p3 --> rotate
	      position["Token position t"] --> angle["Angle for pair i: φ_i = t θ_i<br/>Highlighted pair 0 at m: φ_0 = m θ_0"]
	      frequency["Pair frequency θ_i"] --> angle
	      angle --> rotate["Rotate each pair independently<br/>a_i′ = a_i cos φ_i − b_i sin φ_i<br/>b_i′ = a_i sin φ_i + b_i cos φ_i"]
	      rotate --> slots["Write rotated values into their original slots<br/>a0′, a1′, a2′, a3′, b0′, b1′, b2′, b3′<br/>Still 8 features per head"]
	      slots -->|"Query branch at t = m"| qr["Rotated query q_m′"]
	      slots -->|"Key branch at t = n"| kr["Rotated key k_n′"]
	      qr --> score["Dot product within a head<br/>q_m′ · k_n′<br/>Unscaled attention score"]
	      kr --> score
	      classDef firstPair fill:#cffafe,stroke:#b91c1c,color:#164e63,stroke-width:3px;
	      classDef rotation fill:#ede9fe,stroke:#7c3aed,color:#4c1d95;
	      class p0 firstPair;
	      class angle,rotate rotation;
	  ```
	- **Reshaping is not a learned transformation**: the projections change the values; reshaping only organizes each 16-feature result into two heads. Slot numbers identify coordinates, not numerical feature values.
	- **Pair across halves, then restore the layout**: `(1, 5)`, `(2, 6)`, `(3, 7)`, `(4, 8)` is the convention in these screenshots—not adjacent-slot pairing. Apply the same local pairing separately to every query/key head at every position.
	- **Different positions, same rotation rule**: query pair `i` rotates by `m θ_i`; key pair `i` rotates by `n θ_i`. Their rotated vectors feed the dot product. This diagram stops at the unscaled score; it does not show attention scaling, masking, softmax, or value aggregation.

- ## Initial takeaways
	- **Token identity and position enter at different points**: token embeddings represent the tokens; RoPE incorporates position into attention’s queries and keys.
	- **Do not confuse RoPE with the residual additions**: the two `+` nodes combine each sublayer’s output with its skip connection. They are not positional-embedding additions.
	- **Context length is a model-level claim**: the screenshot’s 1M-token annotation does not mean that applying RoPE alone guarantees useful behavior at that length.

- ## To expand later
	- Why attention needs positional information.
	- Absolute versus relative positional encoding.
	- How RoPE rotates query/key components and encodes relative position.
	- Long-context scaling and extrapolation limits.
