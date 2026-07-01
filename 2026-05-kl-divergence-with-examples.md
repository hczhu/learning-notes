## Expected Surprise
	- For a random variable $X$ with probability distribution $P(X = x) = p_x$, the surprise (or surprisal) is defined as:
	  $$
	  I_P(x) = - \ln p_x
	  $$
	  This is motivated by some simple intuitive constraints we would like to have on any notion of "surprise":
		- An event with probability 1 has no surprise
		- Lower-probability events are strictly more surprising
		- Two independent events are exactly as surprising as the sum of those events' surprisal when independently measured
	- In fact, it's possible to show that these three considerations fix the definition of surprise up to a constant multiple.
	- From this, we have another way of defining entropy, as the expected surprisal of an event:
	  $$
	  H(X) = - \sum_x p_x \ln p_x = \mathbb{E}_P[I_P(X)]
	  $$
	- Now, suppose we (erroneously) believed the true distribution of $X$ to be $Q$, rather than $P$. Then the expected surprise of our model (taking into account that the true distribution is $P$) is:
	  $$
	  \mathbb{E}_P[I_Q(X)] = - \sum_x p_x \ln q_x
	  $$
	- And we now find that:
	  $$
	  D_{KL}(P \parallel Q) = \sum_x p_x (\ln p_x - \ln q_x) = \mathbb{E}_P[I_P(X) - I_Q(X)]
	  $$
	- In other words, KL-divergence is the difference between the expected surprise of your model, and the expected surprise of the correct model (i.e., the model where you know the true distribution $P$). The further apart $Q$ is from $P$, the worse the model $Q$ is for $P$, i.e., the more surprised it should expect to get by reality.
	- Furthermore, this explains why $D_{KL}(P \parallel Q)$ isn't symmetric, e.g., why it blows up when $p_x \gg q_x \approx 0$ but not when $q_x \gg p_x \approx 0$. In the former case, your model is assigning very low probability to an event which might happen quite often, hence your model is very surprised by this. The latter case doesn't have this property, and there's no equivalent story you can tell about how your model is frequently very surprised.
- ## KL divergence as optimal betting payoff
	- Setup
		- Suppose you can bet on the outcome of some casino game, e.g. a version of a roulette wheel with nonuniform probabilities.
		- First, imagine the house is fair, and pays you $1/p_x$ times your original bet if you bet on outcome $x$.
			- This way, any bet has zero expected value:
				- Betting $c_x$ on outcome $x$ means you expect to get:
					- $p_x \times c_x / p_x = c_x$
		- Because the house knows exactly what all the probabilities are, there is no way for you to win money in expectation.
	- Mismatched beliefs (edge)
		- Now imagine the house does not know the true probabilities $P$, but you do.
		- The house's mistaken belief is $Q$.
			- They pay people $1/q_x$ for event $x$, even though the true probability is $p_x$.
		- Since you know more than them, you can profit. The question is: how much?
	- Betting strategy formulation
		- Suppose you have $1 to bet.
		- You bet $c_x$ on outcome $x$:
			- $\sum_x c_x = 1$
		- Let $W$ be your winnings.
		- Use log winnings to model multiplicative growth:
			- Expected log winnings:
				- $$
				  \mathbb{E}[\ln W] = \sum_x p_x \ln \left( \frac{c_x}{q_x} \right)
				  $$
	- Optimization (Lagrangian)
		- Introduce constraint via Lagrangian:
			- $$
			  L(\lambda; B) = \mathbb{E}[\ln W] + \lambda \left( 1 - \sum_x c_x \right)
			  $$
	- Optimal solution
		- The optimal betting strategy:
			- $c_x = p_x$
		- Corresponding expected log winnings:
			- $$
			  \mathbb{E}[\ln W] = \sum_x p_x \ln \left( \frac{p_x}{q_x} \right) = D(P \| Q)
			  $$
	- Interpretation
		- KL divergence $D(P \| Q)$ equals the maximum expected log profit.
		- It quantifies how much you can exploit the house’s incorrect beliefs.
	- Intuition / asymmetry
		- If $p_x \gg q_x$
			- The house overpays for outcome $x$
			- Optimal strategy: bet heavily on $x$
			- Leads to large $D(P \| Q)$
		- If $q_x \gg p_x$
			- No direct way to exploit
			- Only indirect opportunity via other outcomes $y$ where $p_y \gg q_y$
		- This shows KL divergence is asymmetric.