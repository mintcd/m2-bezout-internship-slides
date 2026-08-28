# Speaker Script

Approximate target: 15 minutes, around 1500 spoken words.

## 0:00-0:30 | Title

Good morning. My presentation is about the fine-grained analysis of the Hyperscan engine. The general theme is average-case analysis for modern algorithms. Modern performance is not explained only by classical asymptotic models; it also depends on implementation choices and hardware features.

The way I will present the work is as a mathematical extraction story. We start from a complicated engine, isolate one clean random object, obtain a recurrence, prove a deterministic result, and then return to a stochastic fixed-point problem.

## 0:30-1:30 | Why Hyperscan?

Hyperscan is a high-performance regular expression matching engine. A standard first explanation of regular expression matching is through finite automata. That viewpoint is important, but it does not describe the whole engine. Hyperscan also uses graph decompositions, SIMD literal matchers, filtering and verification phases, and many engineering heuristics.

I am not trying to model every implementation detail. The point is that some implementation choices naturally suggest mathematical objects. In this talk, the object is the length of the longest literal extracted from a random regular expression.

## 1:30-2:45 | From an Engine to a Statistic

Here is the part of the pipeline that motivates the statistic. A regular expression is converted into a Glushkov NFA. Then Hyperscan decomposes the expression into literals and sub-automata. During execution, literals are searched quickly using SIMD string matchers, and the remaining sub-automata are used for verification.

For example, in the expression `ab[0-9]*xy`, the strings `ab` and `xy` are useful literals. The engine can first search for these literals, and only then verify the middle part `[0-9]*`.

The important point is selectivity. Longer literals usually produce fewer candidate matches, so less work remains for verification. This gives the guiding question: under a random model of regular expressions, how long is the longest literal that can be extracted?

## 2:45-4:00 | A Random Model of Regular Expressions

To ask an average-case question, we need a probability distribution on regular expressions. In this work, the model is a BST-like distribution on syntax trees. You can think of it simply as a probability distribution on recursively built expression trees.

There are two parameters. The parameter `u` is the probability of a concatenation node, and `v` is the probability of an alternation node. The remaining probability, `1-u-v`, is the probability of a Kleene star node. At size one, the tree is a literal. For larger sizes, the root operator is chosen according to these probabilities, and the remaining size is split uniformly between subtrees when the operator is binary.

The model is useful because the parameters can be calibrated. For example, on the Snort rule set, maximum-likelihood fitting gives approximately `u = 0.9` and `v = 0.09`. I do not claim this proves the model is realistic in every detail; it just shows that the model has tunable parameters connected to data.

## 4:00-5:30 | The Longest-Literal Recurrence

Let `Y_n` be the length of the longest literal extracted from a random tree of size `n`. The base cases are `Y_1 = 1` for a single letter and `Y_2 = 0` for a Kleene-star root, because the star can generate the empty string and no literal is extracted through that root.

Now condition on the root operator. This is the whole recurrence. For concatenation, literals from the two subtrees can be concatenated, so the contribution is a sum. For alternation, the expression follows one branch or the other, so the contribution is a maximum. For Kleene star, the contribution is zero.

The variable `U_n` is the random split of the remaining size, and the superscripts denote independent copies. So the large object is built from rescaled independent subobjects. This is the same general flavor as many self-similar limiting problems. The question is to understand `y_n = E[Y_n]`, especially whether `y_n ~ C n^alpha`.

## 5:30-6:45 | Bounding the Expectation

Taking expectations gives a first equation for `y_n`. The sum term is easy because expectation is linear. The hard term is the expectation of a maximum. It is not determined only by the two expectations.

The first simplification is to use

`E[max(X,Y)] >= max(E[X],E[Y])`.

This gives a deterministic lower-bound recurrence, denoted by `z_n`. It no longer tracks the full distribution of `Y_n`, but it still contains a max term. So it is a tractable shadow of the stochastic recurrence, while preserving the main technical obstruction.

The deterministic part of the talk studies this sequence `z_n`.

## 6:45-8:10 | Main Deterministic Result

The main deterministic theorem says that, under the condition `u > 1/2`, the lower-bound sequence has exact power-law asymptotics:

`z_n ~ C n^alpha`

for some positive constant `C`. The exponent is determined by

`alpha + v 2^{-alpha} = 2(u+v)-1`.

Here it is important to separate the heuristic from the theorem. The equation for `alpha` comes from the ansatz `z_n ~ C n^alpha`. If we substitute this into the recurrence, the sums become Riemann sums. The sum part gives an integral of `x^alpha`, and the max part gives an integral of `max(x^alpha,(1-x)^alpha)`. Matching the leading power gives the equation.

The theorem then proves that this ansatz is actually correct for the deterministic lower-bound recurrence. The plot shows the numerical picture: after normalizing by `n^alpha`, the ratio stabilizes.

## 8:10-10:45 | Proof Skeleton

I will not go through all details, but I want to explain the shape of the proof.

First, the Riemann-sum estimate identifies what the recurrence does to power functions. This gives the rough estimate `z_n = n^{alpha+o(1)}`.

Next, integral bounds and an induction argument upgrade this to `z_n = Theta(n^alpha)`. So at this stage the exponent is correct up to constants.

The hard part is to prove convergence of `z_n/n^alpha`. The natural goal is to remove the maximum. If the sequence were increasing from the beginning, this would be easy. But the initial conditions are `z_1 = 1` and `z_2 = 0`, and they can create long early fluctuations.

So the proof has to prove eventual monotonicity. First, the consecutive differences vanish: `|Delta z_n| -> 0`. Since `z_n` is unbounded, the sequence eventually dominates all early values. Then a variation estimate controls the remaining oscillations and gives eventual monotonicity.

Once monotonicity is available, the max term can be removed for all large `n`. The recurrence becomes the delayed recurrence displayed here. The final dyadic argument proves that the absolute variations of `z_n/n^alpha` are summable, hence `z_n/n^alpha` converges to a positive constant.

## 10:45-11:45 | Back to the Stochastic Recurrence

Now we return to the original stochastic recurrence. Simulations suggest that the expectation `y_n` also grows like a power, say `C n^{alpha star}`. But the exponent appears to be larger than the deterministic lower-bound exponent.

The reason is exactly where information was lost. The lower bound replaced `E[max(X,Y)]` by `max(E[X],E[Y])`. In genuinely random cases this inequality is strict. The maximum benefits from fluctuations: even if two variables have the same mean, the larger of the two tends to be above that mean.

So the stochastic recurrence is not just a perturbation of the deterministic one. It has a richer limiting object.

## 11:45-13:10 | A Fixed-Point Route

A natural way to study that limiting object is to normalize. Suppose that `Y_n/n^alpha` converges in distribution. Then the limiting random variable must satisfy the fixed-point equation on the slide. The factors `U^alpha` and `(1-U)^alpha` come from the asymptotic sizes of the two subtrees after normalization.

The standard approach would be a contraction method, but there are two obstructions. First, the equation always has the degenerate solution `delta_0`. Second, the operator `T_alpha` does not preserve the mean, so metrics requiring a fixed first moment are difficult to use directly.

The workaround is to let the exponent move with the distribution. For each distribution `mu` with mean one, choose `alpha(mu)` so that applying `T_alpha` preserves the mean. This defines the mean-preserving iteration. The hope is that this iteration identifies the true stochastic exponent.

## 13:10-14:30 | Existence via Tychonoff

The fixed-point existence result uses a compactness argument, in the spirit of a direct method. Define

`Phi(mu) = T_{alpha(mu)}(mu)`.

The goal is to find a compact convex set mapped into itself.

The set used here is `D_2(1,M)`: probability measures on the nonnegative real line with mean one and second moment bounded by `M`. Under `u > 1/2`, one can choose `M` large enough so that `Phi` maps this set into itself. This is the a priori bound.

The remaining ingredients are topological. In the space of bounded Radon measures with the weak topology, this set is nonempty, compact, and convex. Compactness comes from tightness, and the second-moment bound gives uniform integrability, which helps prove weak continuity of `Phi`.

Therefore Tychonoff gives a fixed point `mu star`, with `alpha star = alpha(mu star)`. This proves existence. It is nonconstructive, and it does not yet prove uniqueness or convergence of the iteration.

## 14:30-15:00 | Conclusion

To summarize, Hyperscan motivates mathematical questions because its performance depends on extracted literals and verification. Under a BST-like model, the longest-literal statistic leads to a stochastic recurrence with sums, maxima, and resets.

The main result proved exact asymptotics for a deterministic lower bound. The stochastic recurrence then leads to a fixed-point picture: existence is proved by compactness, while uniqueness, characterization of the true exponent, and convergence remain open.
