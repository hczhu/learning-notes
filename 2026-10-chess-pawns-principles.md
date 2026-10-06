file-created-at:: 2026-10-05

- **Source**: *Winning Chess Strategies* by Yasser Seirawan (with Jeremy Silman) — **Chapter Six, "How to Use Pawns"** (pp. 108–133, Diagrams 65–82, Tests 14–16); pages photographed 2026-10-05.
- **One-liner**: Philidor called pawns **"the soul of chess"** because they are the only men that cannot retreat — so **the pawn structure is the one semi-permanent feature of a position, and therefore it is what dictates the plan**. Every pawn move is an irreversible trade: it gains something and permanently surrenders the squares that pawn used to watch over.
- ## 1. Why pawns determine the plan
	- **1749** — André Philidor announces that pawns are **the soul of chess**. Nobody understands him at the time. **160 years later Emanuel Lasker** explains it:
		- > *"The pawn, being much more stationary than the pieces, is an element of the structure of a position and hence determines the character of the plan appropriate to it."*
	- The point every master now takes for granted: **the placement of the pawns is one of the most important factors in choosing a strategy.** Pieces are fluid and can be re-deployed; pawns mostly cannot. So the pawns are the *terrain*, and you pick a plan to suit the terrain.
	- **The unifying first principle of the whole chapter**: *a pawn move can never be taken back.* Everything below — fixing, restraining, advancing, promoting, majorities, islands — is a consequence of that single asymmetry.
- ## 2. The chapter's maxims, collected
	- | # | Maxim | Section |
	  |---|---|---|
	  | 1 | ***Whenever you can, use your pawns to fix your opponent's pawns.*** | Blocking |
	  | 2 | ***Use your pawns to take away squares from the enemy pieces.*** | Restricting |
	  | 3 | ***Make your pawns work for you in an active way. Don't allow them to sit around like lumps, blocking your own pieces.*** | Advancing |
	  | 4 | ***Don't allow a passed pawn to get blocked. If your opponent does manage to block it, do everything in your power to remove that blockader.*** | Promotion |
	  | 5 | ***Aside from running down the board, pawns also keep enemy pieces out of critical squares. When you push a pawn, be sure you are not handing a juicy square to your opponent on a plate!*** | Drawbacks |
	  | 6 | ***The strengths of a pawn majority are best shown in the endgame.*** | Majorities |
	  | 7 | ***Always attack a pawn chain at its base.*** | Islands |
	  | 8 | ***Avoid creating pawn islands, because lots of pawn islands translates to lots of vulnerable pawn bases.*** | Islands |
	  | 9 | ***Whenever you can, create pawn islands in your opponent's camp. Those islands give you extra weaknesses to attack!*** | Islands |
	- Read 1–2 together and 8–9 together: each pair is **the same idea applied to your opponent and then to yourself.** That symmetry is most of the chapter.
- ## 3. Pawns as blocking and restricting agents
	- Pawns *"cannot swoop down the board in a single bound — they are actually rather ponderous creatures."* What they **can** do is **block other pawns and keep enemy pieces off key squares**. That is their first job, and it costs nothing in tempo once done.
	- **Fixing (maxim 1)** — Diagram 65: White is two pawns up and wants `f4-f5` (freeing his bad Bishop) or `d4-d5` (making a passer). **1...e6!** uses one pawn as a **blocking unit that immobilizes both** White pawns at once; Black follows with `...Ne7` and a Knight to **d5 or f5**, and the win evaporates.
		- The mechanism to remember: **a fixed pawn is a pawn that can no longer run, which makes it a target** — and the square in front of it becomes a permanent outpost.
	- **Restricting (maxim 2)** — Diagram 66 is the same position with three small changes (c-pawn back to c4, h-pawn back to h3, a-pawn moved to g3) and now **White wins**, because `c4` and `g4` **deny the Black Knight d5 and f5**. A sample: if `5...Nc8`, then **6.a5!** keeps the Knight out of b6 — *"a nice example of using a pawn to restrict the activity of an enemy piece."*
		- Same pawns, nearly the same structure, opposite result. **Which squares your pawns deny is the whole evaluation.**
	- **The two Seirawan games are the method applied over 20 moves:**
		- **Van der Wiel–Seirawan, Baden 1980** (Diagram 67): White's Bishop and Knight are blocked by his own `d4` and `f4`; he wants `f4-f5` to free them. Seirawan plays **1...g6!** to stop it dead, then spends the whole game making sure `d4-d5` and `f4-f5` never happen. Once both pawns are permanently frozen, *"they turn out to be targets"* — and he wins by attacking them.
		- **Timman–Seirawan, Lone Pine 1978** (Diagram 68): behind in development, Seirawan plays **1...h5!** to prevent `g2-g4`, which simultaneously **stops White's attack and builds a home on f5 for his Knight.** *"It is very important to get used to this type of restrictive pawn device."*
	- **The licence that makes these weakening moves playable** — stated explicitly both times: **the center is closed.** `...g6` weakens dark squares and `...h5` loosens the kingside, but with a locked center *"his pieces can't get to my King."* Seirawan's own caveat: **"if the center were open, I would want to get my King castled fast!"** (Same structural condition as the closed-position logic in [[2026-10-chess-bishops-principles]] and the King-walk exception in [[2026-10-chess-king-principles]].)
	- **The summary of the whole method**, in his words after Timman: ***"I used my pawns to fix my enemy's pawns on certain squares, turning them into stationary targets. We all know that it is much easier to hit something that cannot run away!"***
- ## 4. The cost of every pawn move (maxim 5)
	- This is the chapter's sharpest single idea, from Diagram 75. A pawn on `c2` currently controls **b3 and d3** — but it has the *potential* to control **b3–b8 and d3–d8**, eleven squares in all.
	  ```
	    pawn on c2   controls now:  b3, d3
	                 potential:     b3 b4 b5 b6 b7 b8
	                                d3 d4 d5 d6 d7 d8
	  
	       1.c3  ->  surrenders b3 and d3        ... forever
	       1.c4  ->  surrenders b3, d3, b4, d4   ... forever
	  
	    "Every time a pawn moves, it loses a bit of that wonderful potential."
	  ```
	- **So every pawn move is a purchase made with squares.** The question is never *"is this push good?"* but ***"is what I gain worth the squares I will never watch again?"***
	- The worked judgment in that position: **1.c4 is good** — it blocks Black's c-pawn (which in turn blocks Black's bad Bishop) and strengthens the grip on the `d5` hole — and giving up d3/d4 is **affordable because the e2-pawn covers those same squares**. But two moves later **2.e4? would be horrible**, permanently conceding `d4` to `...Ne6-d4`; the correct **3.e3!** denies the Knight its support point instead.
	- **The practical test, then: before any pawn push, name the square you are giving up and name the enemy piece that wants it.** If no enemy piece can use it, or another pawn still covers it, push. Otherwise don't.
	- **The same blade cuts both ways** — a pawn can block *the enemy's* pieces or *your own*:
	  ```
	    A pawn move does two things at once:
	  
	      denies squares to          and      denies squares to
	      the ENEMY's pieces   <-- good | bad -->   YOUR OWN pieces
	  
	      Van der Wiel-Seirawan:  ...g6 / ...h5  froze White's d4+f4  -> won
	      Timman-Seirawan:        3.f4?          froze White's OWN
	                                             Bd2 and Nd3          -> lost
	  ```
		- Hence **maxim 3**: *make your pawns work for you in an active way; don't allow them to sit around like lumps, blocking your own pieces.*
- ## 5. Advancing pawns
	- *"Pawns love to advance. Sometimes they will happily jump upon the enemy's swords, sacrificing themselves so that other pieces can live better lives."* Advancing pawns **push other pawns aside, opening files and diagonals and making previously inactive pieces something to be feared** — or they become **runners** heading for promotion.
	- **Pawns as sacrificial lambs** — Diagram 69: White's own `d4` pawn is strangling his whole army (the b2-Bishop and a1-Queen ram into it, the Rooks have no open file, the f3-Knight can't use d4). The answer is not to maneuver around it but to **give it away: 1.d5!**, after which every one of those pieces comes alive.
		- **The lesson**: a pawn is cheap and a dead army is expensive. When a single pawn is the thing blocking three or four of your own pieces, **sacrificing it is often just correct arithmetic**.
- ## 6. Passed pawns and promotion (maxim 4)
	- A pawn's ability to **promote** makes it *"a mighty force in its own right."* When promotion is imminent, *"the very pieces that once snubbed the lowly pawn will desperately try to block its forward progress with their own bodies."*
	- **Knights are the best blockaders** — which is the same conclusion reached from the Knight's side in [[2026-10-chess-knights-principles]].
	- So the strategy is symmetric and concrete: **don't let your passer be blocked; if it is, remove the blockader.**
		- Diagram 72: White breaks the blockade by attacking the blockader itself — **1.Nb5!**, and since Black can't trade (the c-pawn would queen), he must abandon c7; after `1...Na8 2.c7 Nb6 3.Nd6!` and `4.c8=Q Nxc8 5.Nxc8`, White has **won a piece**.
	- **The reframe worth keeping**: a blockaded passer is not a passer, it is a liability — a pawn whose whole value is deferred. Promotion threats only pay when the pawn can actually move.
- ## 7. Pawn majorities
	- **Definition**: more pawns than your opponent in a given area of the board. Their value is that they **usually let you manufacture a passed pawn**.
	- **The folklore and the correction:**
		- Many players prefer a **queenside majority**, on the reasoning that the enemy King usually castles kingside and will be far away from it.
		- But **Alekhine's warning** (on Diagram 77, Yates–Alekhine, The Hague 1921): *"White's celebrated Q-side pawn majority proves to be completely illusory."* And more bluntly: *"one of the most characteristic prejudices of modern chess theory is the widely held opinion that such a pawn majority is important in itself — without any evaluation of the placing of the pieces."*
		- Seirawan's own framing: **don't overrate one majority against another, because King position and the quality of the pawns usually decide who actually wins.** In that game Black's compensation was concrete — *"great freedom of position of the Rook on the only open file"* and *"the dominating position of his active Bishop."*
	- **A majority can be devalued** — Diagram 76: White has 3-v-2 on the queenside, Black 4-v-3 on the kingside, and **whoever moves first wins**. The defensive resource is **1...b6!**: Black's **two** pawns then hold White's **three**, and *"the immobile, devalued majority becomes nothing more than a target."*
		- **The idea in general: a majority that cannot advance is not an asset, it is a cluster of fixed targets.** Compare maxim 1 — devaluing a majority *is* fixing.
	- **Maxim 6 — majorities are an endgame asset.** If you have a healthy majority, **simplify**: *"it does you no good to rave about the wonders of your pawn majority if your opponent can swoop in with his pieces and behead your King!"*
		- **Capablanca's method** (Diagram 78) is the template: trade some pieces, **steer into a simplified position where kingside attacks and hand-to-hand fighting simply cannot happen**, and only then push the pawns for all they are worth.
	- **The reverse-side warning in the same diagram**: Black's queenside majority is *"meaningless at the moment"* because tactics interfere — `1...Nc6? 2.Nxc6 bxc6` turns a mighty majority into **a weak, doubled mess**. A majority is only as good as the pawn structure it survives into.
- ## 8. Pawn chains and pawn islands
	- **Definitions**:
		- A **pawn chain** is a line of connected pawns. Its **base** is the pawn **not protected by another pawn**.
		- A **pawn island** is an isolated group of pawns, or a lone isolated pawn.
	- **Maxim 7 — always attack a pawn chain at its base**, because the base is the only link *"not protected by a pawn, making it vulnerable to attack by enemy pieces."*
	  ```
	    Black attacks the WHITE chain d4-e5:
	  
	        e5   <- the HEAD: defended by d4. Hard nut, leave it.
	        /
	      d4     <- the BASE: defended by NO pawn.  *** aim here ***
	  
	    More pawn islands  =  more bases  =  more places to be attacked.
	  ```
	- **The worked example**: after `1.e4 e6 2.d4 d5 3.e5 c5 4.c3 Nc6 5.Nf3 cxd4 6.cxd4` (Diagram 79), the `e5`-pawn is *"a hard nut to crack"* — so Black ignores it and trains everything on **d4**: `6...Qb6 7.Be2 Nge7 8.Na3 Nf5`.
		- **The payoff is not necessarily winning the pawn.** *"Black may not win this pawn, but he is forcing White's pieces to take up passive positions in order to keep it on the board."* **Tying the defender down is the whole return on the plan.**
	- **Maxims 8 and 9 follow directly**, and they are the same observation twice: **each island has its own base, and every base is a target.** Diagram 80 is the bare-bones case — Black has **one** island (one weak point, f7), White has **three** (b2, f4, h4), *"so White could face some hardship simply because he has three points to defend compared with Black's one."*
	- **Two grandmaster games on manufacturing islands in the opponent's camp:**
		- **Fischer–Trifunović, Bled 1961** (Diagram 81): Fischer must recapture and has two options — `Rxe6`, keeping the pawn count level, or capturing on d4, which **leaves Black with three pawn islands** and a particularly vulnerable e6-pawn. He takes the second, ties Black down to defending it (`4.Re4`), brings the rest of his army up (`5.Be3`), re-attacks, and converts the extra pawn.
		- **Seirawan–Kveinys, Manila 1992** (Diagram 82): he passes over a good space-grabbing plan (`19.Nf4` and `b4`) in favor of pressure on the half-open d-file, specifically **because the islands Black's freeing `...f7-f6` would create would ultimately favor him.** The structural damage was judged worth more than the space.
		- **The shared method**: choose the capture or the pressure that **damages structure** rather than the one that grabs material or space. Structure is permanent; the other two are not.
- ## 9. Takeaways
	- **Pawns are terrain, not just material.** They are the semi-permanent part of the position, so they determine which plans are even available. Evaluate the structure *before* looking for moves.
	- **Every pawn move spends potential that never comes back.** The c2 analysis is the chapter's best single lesson: name the squares you are surrendering and the enemy piece that wants them, *then* decide.
	- **The same pawn move blocks both armies — make sure it blocks theirs.** Seirawan's `...g6`/`...h5` froze his opponent's pieces; Timman's `3.f4?` froze his own. Identical-looking moves, opposite outcomes.
	- **Fixed means hittable.** Fixing an enemy pawn converts it from a mobile unit into a stationary target and hands you the square in front of it. *"It is much easier to hit something that cannot run away."*
	- **A frozen asset is not an asset.** A blockaded passer, an immobile majority, and a pawn that blocks your own bishop are all the same mistake viewed from different angles — value that cannot be cashed.
	- **Attack chains at the base, and count islands before trading.** The number of bases your opponent must defend is a concrete, countable measure of how much of their army will end up passive.
	- **Structure outlasts material and space.** Both Fischer and Seirawan chose the line that damaged the opponent's pawn structure over the one that won the simpler material or territorial point.
	- **Majorities need an endgame and a sober appraisal.** Simplify to cash one in — and remember Alekhine: a majority is worth nothing *"without any evaluation of the placing of the pieces."*
	- Related: [[2026-10-chess-knights-principles]], [[2026-10-chess-bishops-principles]], [[2026-10-chess-rooks-principles]], [[2026-10-chess-king-principles]], [[2026-10-chess-legals-mate-relative-pin]]
