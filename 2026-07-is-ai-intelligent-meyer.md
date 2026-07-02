## Is AI Intelligent? - Bertrand Meyer
- Source
	- Bertrand Meyer, "Is AI Intelligent?", ACM, posted Apr 23 2026
	- DOI: 10.1145/3797898
	- Link: https://bit.ly/3MgwuHz
- Core idea
	- Many debates over whether AI is "intelligent" are not really technical disagreements.
	- They are often definition conflicts between two concepts of intelligence:
		- **Intelligence as understanding**: intelligence means conceptual grasp, rational knowledge, explanation, and "really understanding" what is going on.
		- **Intelligence as coping**: intelligence means adapting to new situations, learning from experience, and producing successful outcomes.
- Why "AI only appears to understand" is weak
	- The claim that an AI system only appears to understand is difficult to test scientifically.
	- A scientific claim should be falsifiable: there should be a reliable experiment that could prove it wrong.
	- If both a human and an LLM can answer many linear algebra questions correctly and both sometimes fail, the output alone does not tell us who "really understands."
	- Arguments such as "my mistakes are superficial, the AI's mistakes prove it has no clue" rely on an undefined inner state.
	- Turing Test and Chinese Room style arguments mostly evaluate outcomes, not an independently measurable essence of understanding.
- The two traditions
	- **European / continental framing**
		- Intelligence is associated with understanding, conceptual knowledge, and explanation.
		- Etymological intuition supports this: Latin *intelligo* means "I understand."
		- This tradition favors deductive reasoning: start from a theory and verify it against facts.
		- Philosophical lineage in the article: Descartes and Kant.
	- **American / Anglo-Saxon framing**
		- Intelligence is associated with adaptation, learning from experience, and successful action.
		- This tradition favors inductive reasoning: start from facts and build theory.
		- Philosophical lineage in the article: Hume, John Stuart Mill, and behaviorists such as Skinner.
- Comparison table
	- | Concept | Intelligence as understanding | Intelligence as coping |
	  |---|---|---|
	  | Core claim | "I am intelligent because I understand." | "I am intelligent because I can act successfully and predict correctly." |
	  | Strength | Elegant, explanatory, conceptually satisfying | Practical, measurable, outcome-oriented |
	  | Weakness | Hard to validate or falsify | Can look like record-keeping or pattern matching rather than "real" intelligence |
	  | AI analogy | Old AI, expert systems, logic-based tools | Modern AI, machine learning, deep learning |
	  | Failure mode | Beautiful explanations with poor prediction | Strong prediction without satisfying explanation |
	  | Scientific standard | Needs a way to test "understanding" | Easier to test by outcomes |
- Old AI vs. modern AI
	- Old AI aligned more with intelligence-as-understanding.
		- Expert systems and logic-based tools tried to encode knowledge explicitly.
		- Meyer's claim: the consensus is that old AI failed.
	- Modern AI aligns more with intelligence-as-coping.
		- Machine learning builds answers to new questions by extrapolating from large bodies of validated past examples.
		- Modern systems are empirical and inductive.
		- Their strength is not that they provide a human-like theory of the world, but that they often produce good results.
- Examples used to challenge "mere next-token prediction"
	- Sentence completion examples can be read as evidence of different kinds of understanding:
		- "I poured myself a cup of ..." suggests co-occurrence and world-pattern understanding.
		- "The capital of France is ..." suggests geographical association.
		- "She unlocked her phone using her ..." suggests semantic relation.
		- "The cat chased the ..." suggests probability over plausible continuations.
		- "If it is raining, I should bring an ..." suggests inference.
	- Meyer's point is not that these prove human-like understanding.
	- The point is that critics must define what kind of understanding is missing and how to test for it.
- Airplane analogy
	- Saying AI is not intelligent because it does not understand in the same way humans do is like saying airplanes do not fly because they do not fly the same way birds do.
	- Different mechanism does not automatically imply absence of the capability.
- Important distinction: explanation vs. prediction
	- Explanatory systems can sound intelligent but fail if they do not predict correctly.
	- Meyer contrasts unfalsifiable grand explanatory theories with scientific theories such as relativity.
	- Relativity did not merely offer a beautiful account of space and time; it made a risky, measurable prediction about light bending during an eclipse.
	- Falsifiability is what separates scientific explanation from rhetorical explanation.
- Questions the article raises
	- Is a translation tool intelligent if it reaches human-quality translation?
	- Is a vibe-coding tool more intelligent than the programmer who uses it?
	- Is a medical-image model more intelligent than a radiologist if it produces fewer false positives and false negatives?
	- Are non-AI systems such as compilers intelligent, given that no human can correctly compile a 100,000-line program by hand in reasonable time?
- My takeaway
	- Arguments about AI intelligence should start by defining intelligence.
	- If intelligence means inner, human-like understanding, the debate needs a falsifiable test for that property.
	- If intelligence means adaptive success, prediction, and useful action, modern AI systems already satisfy many practical criteria.
	- The strongest position is not "AI is intelligent" or "AI is not intelligent" in the abstract.
	- The stronger position is: specify the definition, specify the test, then evaluate the system.
- Useful phrasing
	- "Are we debating intelligence-as-understanding or intelligence-as-coping?"
	- "What experiment would falsify the claim that the system does not understand?"
	- "Different mechanism is not the same as absent capability."
	- "Outcome-based intelligence is easier to measure; understanding-based intelligence is harder to operationalize."
