file-created-at:: 2026-10-04

- **Source**: Chess-book page photographed 2026-10-04 — "Part 1: Tactics and Combinations", Diagram 36 ("White to play"). Book title not visible in the photo; it presents the line as "a famous position from Philidor". Better known as **Légal's Mate** (the Légal trap), after Kermur Sire de Légal, Philidor's teacher.
- **One-liner**: A piece pinned to the *queen* (a relative pin) can still move — Black pins the f3-knight, White moves it anyway and offers the queen, and taking the queen walks into mate.
- ## The moves
	- **1.e4 e5 2.Nf3 d6 3.Nc3 a6?** (a wasted tempo) **4.Bc4 Bg4??** — Black pins the f3-knight to the queen on d1, assuming it cannot move.
	- **5.Nxe5!!** — White breaks the pin and offers the queen.
		- **5...Bxd1? 6.Bxf7+ Ke7 7.Nd5#** — Black wins the queen and is mated.
		- **5...dxe5 6.Qxg4** — the better defense; White is simply a clean pawn up.
- ## Diagram 36 — after 4...Bg4, White to play
	- Uppercase = White, lowercase = Black:
	  ```
	    8  r n . q k b n r
	    7  . p p . . p p p
	    6  p . . p . . . .
	    5  . . . . p . . .
	    4  . . B . P . b .
	    3  . . N . . N . .
	    2  P P P P . P P P
	    1  R . B Q K . . R
	       a b c d e f g h
	  ```
- ## The final position — after 7.Nd5#
	- Three minor pieces deliver mate; Black's own pieces box the king in:
	  ```
	    8  r n . q . b n r
	    7  . p p . k B p p
	    6  p . . p . . . .
	    5  . . . N N . . .
	    4  . . . . P . . .
	    3  . . . . . . . .
	    2  P P P P . P P P
	    1  R . B b K . . R
	       a b c d e f g h
	  ```
	- | Escape square | Why it fails |
	  | --- | --- |
	  | d7 | covered by the **Ne5** |
	  | e8, e6 | covered by the **Bf7** |
	  | f6 | covered by the **Nd5** |
	  | f7 (capture the bishop) | the bishop is defended by the **Ne5** |
	  | d6, f8 | occupied by **Black's own** pawn and bishop |
	  | Capture the checking knight | only the queen could, and **Black's own d6 pawn** blocks d8–d5 |
- ## The lesson
	- **A relative pin is not an absolute pin.** A piece pinned to the queen may legally move; it only costs the queen if the opponent can afford to take it. Here White can, because taking the queen allows mate.
	- The book's moral: *never assume the pinned piece won't move.*
	- **As Black**: before pinning a knight to the queen, ask what that knight captures if it moves anyway. Here e5 was defended only by ...d6xe5. Developing (e.g. 4...Nf6) instead of 3...a6 / 4...Bg4 avoids the problem.
	- **As White**: against 5...dxe5 the reward is a pawn, not mate — the mate needs Black to grab the queen.
- ## Caveat — when the sacrifice is unsound
	- The trap relies on e5 being defended only by the d6-pawn. **With a Black knight on c6, 5.Nxe5? is unsound**: 5...Nxe5! recaptures with the knight while the bishop still attacks the queen, and White loses material.
	- That was the setup in the original 1750 game, **Légal vs. Saint Brie**: 1.e4 e5 2.Bc4 d6 3.Nf3 Nc6 4.Nc3 Bg4 5.Nxe5? Bxd1?? 6.Bxf7+ Ke7 7.Nd5# — mate only because Black took the queen.
	- The book's move order (3...a6, no knight on c6) is the version where the sacrifice actually works.
