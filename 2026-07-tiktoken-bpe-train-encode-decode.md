- **Source**: `tiktoken/_educational.py` in [openai/tiktoken](https://github.com/openai/tiktoken), commit `08a5f3b` (2026-05-24). A ~220-line pure-Python mirror of the Rust core, written to be read rather than run in production. All numbers below were measured by running it, not read off the page.
- **One-liner**: BPE training picks merges by **frequency**; BPE encoding picks merges by **rank**. Frequency is consulted exactly once — at training time — and then frozen into the token ids, because a new token's id *is* `len(vocab)` at the moment it was learned. Encoding is therefore not a greedy search for the shortest output; it is a **replay of training history in the order it happened**.

- ## Orientation
	- Three operations, deliberately asymmetric in difficulty: `bpe_train()` learns a vocabulary (expensive, once), `bpe_encode()` turns bytes into ids (search, per request), `decode_bytes()` turns ids back into bytes (a dict lookup, nearly free).
	- Two entry points: `SimpleBytePairEncoding.train(data, vocab_size, pat_str)` learns a fresh vocab, while `SimpleBytePairEncoding.from_tiktoken("cl100k_base")` borrows a **real production vocabulary** and runs the same encode loop over it — the algorithm below is literally what runs on GPT-4's vocab.
	- What is *not* here, and lives in `tiktoken/core.py` instead: special tokens (`<|endoftext|>`), `allowed_special` handling, and unstable-token logic.

- ## Step 0 — the regex pre-split, shared by train and encode
	- Before any BPE runs, text is chopped into "words" by a regex — training at `_educational.py:136`, encoding at `_educational.py:30`. Same pattern, same role.
	- The GPT-2 pattern: `'s|'t|'re|'ve|'m|'ll|'d| ?[\p{L}]+| ?[\p{N}]+| ?[^\s\p{L}\p{N}]+|\s+(?!\S)|\s+`
	- **Merges can never cross a split boundary.** BPE only ever runs *inside* one piece. This single fact explains most "why did it tokenise that way?" surprises — they are the regex's doing, not BPE's.
	- Hence a leading space attaches to the *following* word: `regex.findall(pat, "hello world")` → `['hello', ' world']`. So `" world"` is one token candidate, and `"world"` after a newline is a **different token entirely**.

- ## Training — `bpe_train()` (`_educational.py:119`)
	- **The floor**: ids `0..255` are pre-assigned to the 256 single-byte values before anything is learned (`_educational.py:126`). This is why `vocab_size` must be ≥ 256, and why **there is no `UNK` token** — any byte sequence on earth is encodable, worst case one token per byte.
	- **The loop**, while the vocab is smaller than the target:
		- **Count** every adjacent pair across all words into a `Counter`.
		- **Pick** the single most frequent pair.
		- **Mint** a new token: bytes are `pair[0] + pair[1]`, and critically the id is `len(ranks)`.
		- **Rewrite** the entire corpus, replacing that pair everywhere, so the next iteration counts pairs *of the merged units*.
	- **The invariant everything downstream depends on**: because the id is `len(ranks)`, **id == rank == the order the merge was learned**. A token's number is not arbitrary — it is a timestamp. Lower id means learned earlier, which means it was more frequent at the time.
	- **What it actually learns** (trained on the file's own source, `vocab_size=600`): final vocab is `600 = 256 bytes + 344 merges`, and the first eight merges in order are `b'  '`, `b'    '`, `b'   '`, `b'in'`, `b'en'`, `b'\n    '`, `b'or'`, `b' t'`.
		- Whitespace dominates because the corpus is indented Python source. **A vocabulary is a fingerprint of its corpus**, nothing more.
		- Note the ordering quirk: 4 spaces (rank 257) is learned *before* 3 spaces (rank 258). Once `b'  '` exists, a 4-space indent is the pair `(b'  ', b'  ')` — an adjacency 3 spaces cannot form. **Merges compound on previous merges.**
	- **Rewrite-loop details** (`_educational.py:159`): `while i < len(word) - 1` walks pairs left to right, consuming two pieces on a match and one otherwise; the trailing `if i == len(word) - 1` catches the final unpaired piece and also handles a one-piece word, where the loop body never runs. Matching is **non-overlapping and leftmost-first** — merging `(a,a)` in `[a,a,a]` gives `[aa, a]`, not `[a, aa]`.
	- **Gotchas**
		- **It crashes on small corpora.** If the data runs out of mergeable pairs before `vocab_size` is reached, `max()` receives an empty `Counter`: `bpe_train(data="aaa", vocab_size=300, ...)` → `ValueError: max() iterable argument is empty`.
		- **Ties are broken arbitrarily** — `max()` returns the first maximum in `Counter` insertion order. Deterministic for fixed input, but not principled.
		- **Everything is recounted every iteration** — no incremental pair statistics, so cost is roughly `O(vocab_size × corpus_size)`. Real trainers maintain an indexed priority queue.
		- **`SimpleBytePairEncoding.train()` cannot silence the visualiser** — it does not forward `visualise` to `bpe_train` (`_educational.py:71`), which defaults to `"colour"`. Call `bpe_train(...)` directly with `visualise=None` to train quietly.

- ## Encoding — `bpe_encode()` (`_educational.py:83`)
	- **The algorithm**: start maximally fragmented, one part per byte. Then repeat until no adjacent pair exists in the vocabulary — scan **all** adjacent pairs, keep the one whose merged bytes have the **lowest rank** (not the longest, not the leftmost), and apply **exactly one** merge (`_educational.py:110`) before rescanning from scratch. Finally map each surviving part to its id, a lookup that can never fail because all 256 single bytes are present.
	- **Why lowest rank is the whole trick**: rank is learning order. Training built these tokens in sequence, each merge assuming the previous ones were already applied, so encoding must replay that history **in the same order**. It is a consistency requirement, *not* a heuristic for shortness — see the subsection below for what breaks otherwise.
	- Merge-by-merge trace for `"hello world"`, where `␣` is a literal space (verified with `visualise="simple"`):
	- | Pass | `hello` | `" world"` |
	  |---|---|---|
	  | 0 — raw bytes | `h e l l o` | `␣ w o r l d` |
	  | 1 | `he l l o` | `␣ w or l d` |
	  | 2 | `he ll o` | `␣w or l d` |
	  | 3 | `he llo` | `␣w or ld` |
	  | 4 | `hello` | `␣wor ld` |
	  | 5 | — | `␣world` |
		- Both words collapse to a **single** token: `encode("hello world")` → `[399, 389]`, with `ranks[b'hello'] == 399` and `ranks[b' world'] == 389`.
		- The trace reads bottom-up as a merge tree being rebuilt — a good mental picture of BPE as **reconstruction rather than lookup**.
	- **Properties**: greedy and **not optimal** — it minimises nothing globally and does not guarantee the fewest tokens. Ties go leftmost (the comparison is a strict `rank < min_rank`). Cost is `O(n²)` per word, since each full rescan yields one merge; acceptable only because the regex pre-split caps word length at ~20 bytes.

	- ### Why lowest rank, not highest
		- The natural question is what breaks if the scan keeps the **highest**-ranked mergeable pair instead. Swapping `min` for `max` is a one-character change, and it is instructive because the result is not a crash but a quietly much worse tokeniser.
		- **Worked example — the word `" merges"`** (`␣` is a literal space). At the very first pass four merges are available, and the two strategies disagree immediately:
		- | Candidate pair | Merged bytes | Rank |
		  |---|---|---|
		  | `␣` + `m` | `␣m` | 282 |
		  | `m` + `e` | `me` | **418** |
		  | `e` + `r` | `er` | 284 |
		  | `g` + `e` | `ge` | 295 |
		- | Pass | lowest-rank (canonical) | highest-rank |
		  |---|---|---|
		  | 0 — raw bytes | `␣ m e r g e s` | `␣ m e r g e s` |
		  | 1 | `␣m` (282) → `␣m e r g e s` | `me` (418) → `␣ me r g e s` |
		  | 2 | `er` (284) → `␣m er g e s` | `ge` (295) → `␣ me r ge s` |
		  | 3 | `ge` (295) → `␣m er ge s` | *stuck* |
		  | 4 | `erge` (312) → `␣m erge s` | — |
		  | 5 | `␣merge` (326) → `␣merge s` | — |
		  | 6 | `␣merges` (541) → `␣merges` | — |
		  | **Result** | **1 token** | **5 tokens** |
			- The damage is done in pass 1. Picking `me` (rank 418) consumes the `m` that `␣m` needed *and* the `e` that `er` needed, stranding the `r` between them. None of `␣me`, `mer`, `rge` or `ges` were ever learned, so the word dead-ends three merges short of the single token that exists for it.
		- **Why this is systematic, not bad luck**: the vocabulary is a merge *tree* — `b' merges'` exists only because `b' merge'` and `b's'` were built first, and so on down. Lowest-first walks that tree bottom-up in construction order and therefore always reaches the root. Highest-first grabs a late, rare, mid-level node early; that node is not a child of anything the surviving pieces can combine with, so the chain dead-ends. Since rank is inverse frequency, "highest rank" literally means **prefer the rarest merge available** — exactly backwards for a compressor.
		- **Measured over the whole file** (`vocab_size=600`): **2674 tokens lowest-rank vs 3290 highest-rank, +23.0%**, with 92 of 452 unique words segmented differently. Worst offenders:
		- | Word | lowest-rank | highest-rank |
		  |---|---|---|
		  | `' SimpleBytePairEncoding'` | 1 token | 10 tokens |
		  | `' character'` | 1 token | 6 tokens |
		  | `' merges'` | 1 token | 5 tokens |
		  | `' replacement'` | 2 tokens | 7 tokens |
		- **The dangerous part is that it fails silently.** Decoding is pure concatenation and therefore order-agnostic, so the round-trip is still byte-exact — `b"".join(parts) == original` holds, and every `assert decode(encode(x)) == x` still passes. Nothing in the tokeniser's own tests would flag it. The damage lands downstream: a model trained on canonical segmentations would receive `' character'` as 6 tokens it has effectively never seen in that arrangement, degrading output quality with no error anywhere.
		- Worth keeping in proportion: lowest-first is not *optimal* either. `' implementation'` encodes to 4 tokens (`b' i'`, `b'mple'`, `b'ment'`, `b'ation'`) and a shorter split may well exist. Its job was never to minimise token count — it is to **reproduce the trainer's segmentation exactly**, and that reproducibility is precisely what highest-first throws away.

- ## Decoding — `decode_bytes()` / `decode()` / `decode_tokens_bytes()` (`_educational.py:39`)
	- The inverse table is built once in `__init__` (`_educational.py:20`) by flipping `mergeable_ranks`. `decode_bytes` is then the entire algorithm: `b"".join(self._decoder[t] for t in tokens)` — look up, concatenate. `decode()` wraps that with `.decode("utf-8", errors="replace")`; `decode_tokens_bytes()` skips the join to preserve token boundaries for visualisation.
	- **Why decoding is trivial but encoding is not**: a token already *is* its bytes. The id → bytes map is total and injective, so decoding resolves no ambiguity and needs one `O(n)` pass. Encoding is hard because many segmentations of the same byte string exist and exactly one matches what training would have produced.
	- **The UTF-8 caveat**: token boundaries are **byte** boundaries, not character boundaries, so a multi-byte character can be split across two tokens. An individual token may therefore decode to `�` — only the *concatenation* is guaranteed valid. This is why `decode()` passes `errors="replace"`, and why `visualise_tokens()` carries the same warning at `_educational.py:190`.
		- Practical rule: **never decode tokens one at a time and join the strings**. Join the bytes, then decode once.

- ## The asymmetry in one table
	- | Phase | Selects by | Mechanism | Cost |
	  |---|---|---|---|
	  | **Train** | frequency — `argmax count(pair)` | mint id `= len(ranks)`, rewrite corpus | `O(vocab × corpus)` |
	  | **Encode** | rank — `argmin rank(pair)` | replay merge history, one merge per scan | `O(n²)` per word |
	  | **Decode** | nothing | `id → bytes`, concatenate | `O(n)` |
		- Conflating the first two columns is the usual source of confusion. Frequency never reappears at encode time; the **ids themselves carry the ordering**.

- ## Gotchas found while running it
	- **The docstrings are stale.** `encode()` claims `[388, 372]` (`_educational.py:27`); the actual result today is `[399, 389]`. Not a bug — `train_simple_encoding()` trains on the file's *own source* via `open(__file__)`, so every edit to `_educational.py` changes the corpus, the merges, and every id. A self-referential vocabulary.
	- **`train_simple_encoding()` fails in a `python3 -i` REPL** with `NameError: name '__file__' is not defined`. CPython sets `__file__` while a script runs, then **deletes it from `__main__`** before handing control to the interactive prompt; the function reads `__file__` at call time, by which point it is gone.
		- Fix — import instead of running as a script, so the lookup resolves in the module's own globals: `python3 -i -c "from tiktoken._educational import *"`. Also works: `python3 -im tiktoken._educational`, since `runpy` does not delete the attribute.
	- **`__file__` is not guaranteed to exist at all** — it is absent under `python3 -c`, absent on builtin modules, `None` on namespace packages, and the literal string `'<stdin>'` (a path that does not exist) when a script is piped in. Code that needs its own source should capture `__file__` at import time rather than read it at call time.

- ## Reproduce
	- ```python
	  from tiktoken._educational import bpe_train, SimpleBytePairEncoding
	  gpt2 = r"""'s|'t|'re|'ve|'m|'ll|'d| ?[\p{L}]+| ?[\p{N}]+| ?[^\s\p{L}\p{N}]+|\s+(?!\S)|\s+"""
	  data = open("tiktoken/_educational.py").read()

	  ranks = bpe_train(data=data, vocab_size=600, pat_str=gpt2, visualise=None)  # quiet
	  merges = [b for b, r in sorted(ranks.items(), key=lambda kv: kv[1]) if r >= 256]
	  print(len(ranks), len(merges), merges[:8])

	  enc = SimpleBytePairEncoding(pat_str=gpt2, mergeable_ranks=ranks)
	  enc.encode("hello world", visualise="simple")   # watch the merges replay
	  ```
		- Building tiktoken from source needs a Rust toolchain (`rustup`), plus `rustup target add x86_64-apple-darwin` if the Python interpreter is x86_64 while the machine is Apple Silicon.
		- Swapping in a production vocabulary is one line — `SimpleBytePairEncoding.from_tiktoken("cl100k_base")` — but it downloads the vocab on first call, so it needs network and a working `requests` import.
