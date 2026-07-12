- **Source**: 崔添翼 (Tianyi Cui), *背包问题九讲 2.0 beta1.2* ("Nine Lectures on Knapsack Problems"), 2012-05-08, part of the *动态规划的思考艺术* (The Art of Dynamic Programming Thinking) series. CC BY-NC-SA. https://github.com/tianyicui/pack
- **One-liner**: Every knapsack variant reduces to one core DP recurrence $F[i,v]=\max\{F[i-1,v],\ F[i-1,v-C_i]+W_i\}$; the variants differ only in loop order, item preprocessing (splitting/grouping), or extra state dimensions.

- ## 1. 0/1 Knapsack (the foundation)
	- **Problem**: $N$ items, knapsack capacity $V$. Item $i$ has cost $C_i$ and value $W_i$. Each item can be taken **at most once**. Maximize total value.
	- **State**: $F[i,v]$ = max value using the first $i$ items with capacity $v$.
	- **Recurrence** (the master equation — nearly all knapsack equations derive from it):
	  $$
	  F[i,v] = \max\{F[i-1,v],\ F[i-1,v-C_i]+W_i\}
	  $$
	  The two terms = the two strategies: skip item $i$, or take item $i$ (reducing to the subproblem with capacity $v-C_i$).
	- **Space optimization**: $O(VN)$ time is optimal, but space drops from $O(VN)$ to $O(V)$ with a 1-D array — iterate $v$ **downward** ($V \to C_i$) so that $F[v-C_i]$ still holds the previous row's value $F[i-1, v-C_i]$:
	  ```
	  def ZeroOnePack(F, C, W):
	      for v = V downto C:
	          F[v] = max(F[v], F[v-C] + W)
	  ```
	- **Initialization trick** (two problem flavors):
		- "Exactly fill the knapsack": init $F[0]=0$, $F[1..V]=-\infty$ (only capacity-0 is a legal empty state).
		- "At most $V$" (no fill requirement): init $F[0..V]=0$ (empty set is legal at every capacity).
	- **Constant optimization**: the inner loop lower bound can be tightened to $\max(V - \sum_{j\ge i} W_j,\ C_i)$ when only $F[V]$ is needed.

- ## 2. Complete (Unbounded) Knapsack
	- **Problem**: same, but each item type has **unlimited supply**.
	- Naive recurrence $F[i,v]=\max\{F[i-1,v-kC_i]+kW_i \mid 0 \le kC_i \le v\}$ costs $O(NV\Sigma\frac{V}{C_i})$ — too slow.
	- **Equivalent transformed recurrence**:
	  $$
	  F[i,v] = \max\{F[i-1,v],\ F[i,v-C_i]+W_i\}
	  $$
	  Note the second term uses row $i$ (may re-take the same item), not $i-1$.
	- **$O(VN)$ algorithm**: identical 1-D code to 0/1, but iterate $v$ **upward** ($C_i \to V$) — this deliberately allows $F[v-C_i]$ to already include item $i$:
	  ```
	  def CompletePack(F, C, W):
	      for v = C to V:
	          F[v] = max(F[v], F[v-C] + W)
	  ```
	- **Key insight**: loop direction is the entire difference between 0/1 (downward = each item once) and unbounded (upward = item reusable).
	- **Simple pruning**: if $C_i \le C_j$ and $W_i \ge W_j$, item $j$ is dominated and can be discarded ($O(N^2)$, or $O(V+N)$ with counting-sort by cost). Helps in practice, doesn't improve worst case.
	- **Binary splitting** (also converts to 0/1): split item $i$ into items with cost $C_i 2^k$, value $W_i 2^k$ for all $k$ with $C_i 2^k \le V$ — any count is a sum of powers of 2, giving $O(\log\lfloor V/C_i \rfloor)$ items per type.

- ## 3. Bounded (Multiple) Knapsack
	- **Problem**: item $i$ available at most $M_i$ times.
	- Naive: $F[i,v]=\max\{F[i-1,v-kC_i]+kW_i \mid 0 \le k \le M_i\}$, cost $O(V\Sigma M_i)$.
	- **Binary splitting trick**: split item $i$ into 0/1 items with multiplier coefficients $1, 2, 4, \dots, 2^{k-1}, M_i-2^k+1$ (where $k$ is the largest integer with $M_i - 2^k + 1 > 0$). Every count $0..M_i$ — and no count $> M_i$ — is representable as a subset sum of these coefficients.
		- Example: $M_i = 13$ splits into coefficients $1, 2, 4, 6$.
		- Complexity improves to $O(V \Sigma \log M_i)$.
	  ```
	  def MultiplePack(F, C, W, M):
	      if C * M >= V:            # supply effectively unlimited
	          CompletePack(F, C, W); return
	      k = 1
	      while k < M:
	          ZeroOnePack(F, k*C, k*W)
	          M -= k; k *= 2
	      ZeroOnePack(F, C*M, W*M)   # remainder
	  ```
	- **Feasibility-only version in $O(VN)$**: if only asking "can capacity $v$ be exactly filled" (no values), define $F[i,j]$ = max leftover count of item $i$ after filling capacity $j$ with the first $i$ items ($-1$ = infeasible). Each state updates in $O(1)$. (A monotone-deque optimization achieves $O(VN)$ for the general valued case too.)

- ## 4. Mixed Knapsack
	- Items are a mix of 0/1, unbounded, and bounded types. Because each item type has its own self-contained update procedure, just dispatch per item:
	  ```
	  for i = 1 to N:
	      if item i is 0/1:        ZeroOnePack(F, C_i, W_i)
	      elif item i is unbounded: CompletePack(F, C_i, W_i)
	      else:                     MultiplePack(F, C_i, W_i, M_i)
	  ```
	- **Lesson**: abstracting each variant into a per-item procedure makes "hard" composite problems trivially decomposable — the power of abstraction.

- ## 5. Two-Dimensional Cost Knapsack
	- **Problem**: each item has two costs $C_i$, $D_i$ with capacities $V$, $U$. Add one state dimension:
	  $$
	  F[i,v,u] = \max\{F[i-1,v,u],\ F[i-1,v-C_i,u-D_i]+W_i\}
	  $$
	- Space: 2-D array suffices; loop $v, u$ downward for 0/1 items, upward for unbounded, split for bounded — same rules as 1-D.
	- **Common disguised form**: "take at most $U$ items total" is just a second cost dimension where every item costs 1.
	- **Perspective**: a 2-D knapsack is a knapsack over the domain of pairs of non-negative integers — all 1-D techniques carry over with the number domain enlarged.
	- **General lesson**: when a familiar DP problem gains a new constraint, add a state dimension for it.

- ## 6. Grouped Knapsack
	- **Problem**: items partitioned into $K$ groups; at most **one item per group**.
	- **Recurrence** ($F[k,v]$ = max value using the first $k$ groups):
	  $$
	  F[k,v] = \max\{F[k-1,v],\ F[k-1,v-C_i]+W_i \mid \text{item } i \in \text{group } k\}
	  $$
	- 1-D implementation — loop order matters: **group → capacity (downward) → items in group**, which guarantees at most one item per group is chosen:
	  ```
	  for k = 1 to K:
	      for v = V downto 0:
	          for item i in group k:
	              F[v] = max(F[v], F[v-C_i] + W_i)
	  ```
	- Many knapsack variants reduce to grouped knapsack; it also leads to the "generalized item" concept.

- ## 7. Dependent Knapsack (tree knapsack)
	- **Problem**: item $i$ may depend on item $j$ (taking $i$ requires taking $j$). Simplified version (from NOIP2006 "金明的预算方案"): dependencies form stars — a "main item" with attachments; no chains.
	- **Key idea**: a main item + its attachment set has exponentially many strategies ($2^n+1$), but they are mutually exclusive → treat them as **one group**. To avoid enumerating all subsets, first run a 0/1 knapsack over just the attachments to get $F_k[0..V-C_k]$ (best attachment value at each cost); this collapses the group to $V-C_k+1$ candidate items (cost $v$, value $F_k[v-C_k]+W_k$). Then solve as grouped knapsack.
	- **General version**: dependencies form a forest → **tree DP**: compute each node's attachment-set knapsack bottom-up (children first), converting each subtree into a generalized item. Solving a parent requires DP results of its children.

- ## 8. Generalized Items (泛化物品)
	- **Definition**: an item whose value depends on the cost allocated to it — a function $h(v)$ on $v \in 0..V$ (equivalently an array $h[0..V]$): allocate cost $v$, receive value $h(v)$.
		- A 0/1 item with cost $c$, value $w$: $h(c)=w$, else 0.
		- An unbounded item: $h(v)=w\cdot\frac{v}{c}$ when $c \mid v$, else 0. A bounded item additionally requires $\frac{v}{c} \le m$.
		- A whole item **group** is one generalized item: $h(v)$ = best value in the group at cost exactly $v$.
	- **Sum of generalized items** (the core operation):
	  $$
	  f(v) = \max\{h(k) + l(v-k) \mid 0 \le k \le v\}
	  $$
	  The sum $f$ is itself a generalized item; computing it costs $O(V^2)$. (This is the $(\max,+)$ convolution.)
	- **Unifying theory**: solving *any* knapsack problem = computing the generalized item corresponding to the whole problem, typically by expressing it as a sum of simpler generalized items. Replacing two items with their sum never changes the answer; the final answer is $\max_v s(v)$ over the total sum $s$.

- ## 9. Variations in What Is Asked
	- **Min instead of max** ("minimum total value/count"): swap max → min in the recurrence (adjust init accordingly).
	- **Reconstructing the solution**: record which branch produced each $F[i,v]$ (array $G[i,v] \in \{0,1\}$), then walk backwards from $F[N,V]$; or recompute the branch on the fly by testing $F[i,v]=F[i-1,v]$ vs $F[i,v]=F[i-1,v-C_i]+W_i$ (no extra array needed).
	- **Lexicographically smallest optimal solution**: relabel items $x \gets N+1-x$, run the standard DP, and when both branches tie during reconstruction, prefer taking the item.
	- **Counting solutions**: replace max with **sum**:
	  $$
	  F[i,v] = \mathrm{sum}\{F[i-1,v],\ F[i,v-C_i]\}, \quad F[0,0]=1
	  $$
	  Valid because the recurrence already enumerates every distinct composition exactly once.
	- **Counting optimal solutions**: track $F[i,v]$ (max value) and $G[i,v]$ (count of ways achieving it) together; add counts from whichever branch(es) attain the max.
	- **K-th best solution**: promote each state from a scalar to a **sorted list of the top $K$ values**: $F[i,v,1..K]$. Each max in the recurrence becomes a merge of two sorted lists (each merge $O(K)$), total $O(VNK)$. Correctness: a correct recurrence enumerates all strategies; keeping the top $K$ per state (instead of only the top 1) preserves the $K$ best overall. Watch the problem's definition of "distinct" — if equal-value distinct strategies count as one solution, dedupe within each list.

- ## Cheat Sheet
	- | Variant | Recurrence / trick | Complexity |
	  | --- | --- | --- |
	  | 0/1 | 1-D array, $v$ loops **down** | $O(VN)$ time, $O(V)$ space |
	  | Unbounded | 1-D array, $v$ loops **up** | $O(VN)$ |
	  | Bounded | binary-split into 0/1 items | $O(V\Sigma\log M_i)$; $O(VN)$ w/ monotone deque |
	  | Mixed | dispatch per-item procedure | per component |
	  | 2-D cost | add a state dimension | $O(VUN)$ |
	  | Grouped | ≤1 per group; loop group→$v$↓→items | $O(V\Sigma\|\text{group}\|)$ |
	  | Dependent | attachments → inner 0/1 → group; forests → tree DP | — |
	  | Generalized items | $(\max,+)$ convolution of value functions | $O(V^2)$ per sum |

- ## Meta-lessons (author's takeaways)
	- Understand *how* each recurrence is derived, not just what it is — deriving equations yourself is how DP skill is built.
	- Hard problems are simple problems stacked; solid grasp of the three basic knapsacks lets you decompose composite ones.
	- Abstraction (per-item update procedures) makes mixed problems trivial.
	- The author failed NOIP2006's dependent-knapsack problem with a 100+ line program scoring 0, later distilled the "group + dependency" insight — "failure is nothing shameful; gaining nothing from failure is."
