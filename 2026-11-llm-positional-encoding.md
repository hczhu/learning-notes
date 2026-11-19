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

- ## Initial takeaways
	- **Token identity and position enter at different points**: token embeddings represent the tokens; RoPE incorporates position into attention’s queries and keys.
	- **Do not confuse RoPE with the residual additions**: the two `+` nodes combine each sublayer’s output with its skip connection. They are not positional-embedding additions.
	- **Context length is a model-level claim**: the screenshot’s 1M-token annotation does not mean that applying RoPE alone guarantees useful behavior at that length.

- ## To expand later
	- Why attention needs positional information.
	- Absolute versus relative positional encoding.
	- How RoPE rotates query/key components and encodes relative position.
	- Long-context scaling and extrapolation limits.
