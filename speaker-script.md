# Speaker Script

Approximate target: 15 minutes, around 1500 spoken words.

## 0:00-0:30 | Title

Good morning. My presentation is about the fine-grained analysis of the Hyperscan engine. The general theme is average-case analysis for modern algorithms. In this setting, classical asymptotic models are often not enough, because practical performance also depends on implementation choices and hardware features. I will focus on one question that can be isolated mathematically: the length of the longest literal extracted from a random regular expression.

## 0:30-1:30 | Why Hyperscan?

Hyperscan is a high-performance regular expression matching engine. From a textbook point of view, regular expression matching is usually explained using finite automata. That is important, but it does not describe the whole engine. Hyperscan also uses graph decompositions, SIMD literal matchers, filtering and verification phases, and many engineering heuristics.

This makes Hyperscan interesting from two sides. On the systems side, its speed comes from hardware-aware implementation. On the mathematical side, some implementation choices create clean probabilistic questions. I will focus on one tractable statistic connected to how the engine works.

## 1:30-2:45 | From an Engine to a Statistic

Here is the part of the compilation pipeline that motivates the statistic. A regular expression is first converted into a Glushkov NFA. Then Hyperscan applies graph analyses, especially through Violet and Rose, to decompose the expression into literals and sub-automata. During execution, literals are searched quickly using SIMD string matchers, and the remaining sub-automata are used for verification.

For example, in the expression `ab[0-9]*xy`, the strings `ab` and `xy` are useful literals. Hyperscan can first search for these literals, and only then verify the middle part `[0-9]*`. Longer literals are usually more selective: they produce fewer candidate matches, so less work remains for verification.

This leads to the guiding question: under a random model of regular expressions, how long is the longest literal that can be extracted?

## 2:45-4:00 | A Random Model of Regular Expressions

To ask an average-case question, we need a probability distribution on regular expressions. I use a BST-like distribution on syntax trees. It is not meant to reproduce every detail of real datasets, but it is flexible and tractable.

There are two parameters. The parameter `u` is the probability of a concatenation node, and `v` is the probability of an alternation node. The remaining probability, `1-u-v`, is the probability of a Kleene star node. At size one, the tree is just a literal. For larger sizes, the root operator is chosen according to these probabilities, and the remaining size is split uniformly between the subtrees when the operator is binary.

The parameters can also be fitted to data. In the Snort rule set, maximum-likelihood fitting gives approximately `u = 0.9` and `v = 0.09`, so concatenation is dominant and alternation is present but less frequent.

## 4:00-5:30 | The Longest-Literal Recurrence

Let `Y_n` be the length of the longest literal extracted from a random tree of size `n`. The base cases are natural: `Y_1 = 1` for a single letter, and `Y_2 = 0` for a Kleene-star root, because in this model the star can generate the empty string and no literal is extracted through that root.

Now condition on the root operator. For concatenation, literals from the two subtrees can be concatenated, so the contribution is a sum. For alternation, the expression follows one branch or the other, so the contribution is a maximum. For Kleene star, the contribution is zero.

The variable `U_n` represents the random split of the remaining size, and superscripts denote independent copies. This gives the distributional recurrence on the slide. The problem is to understand `y_n = E[Y_n]`, especially whether `y_n ~ C n^alpha`.

## 5:30-6:45 | Bounding the Expectation

Taking expectations gives a first equation for `y_n`. The sum term is easy, because expectation is linear. The difficult term is the expectation of a maximum. It is not determined only by the expectations of the two variables.

The first simplification is to use the inequality `E[max(X,Y)] >= max(E[X],E[Y])`. This gives a deterministic lower-bound recurrence, denoted by `z_n`. This recurrence no longer keeps track of the whole distribution of `Y_n`, but it still keeps a max term, so it is not a trivial history-sum recurrence.

The deterministic part of the talk studies this sequence `z_n`. It is a lower bound, but it still captures sums, maxima, and resets.

## 6:45-8:10 | Main Deterministic Result

The main deterministic theorem says that, under the condition `u > 1/2`, this lower-bound sequence has exact power-law asymptotics:

`z_n ~ C n^alpha`

for some positive constant `C`. The exponent is not arbitrary. It is determined by the equation

`alpha + v 2^{-alpha} = 2(u+v)-1`.

This equation comes from a power-law ansatz. If we pretend that `z_n` behaves like `C n^alpha` and substitute this into the recurrence, the sums become Riemann sums. The sum part gives the integral of `x^alpha`, while the max part gives the integral of `max(x^alpha,(1-x)^alpha)`. Matching the leading power of `n` gives exactly this equation.

The plot shows the numerical evidence: after normalizing by `n^alpha`, the ratio stabilizes, consistent with convergence to a constant.

## 8:10-10:45 | Proof Skeleton

The proof is organized as a bootstrap. The first lemma is a Riemann-sum estimate, identifying the leading effect of applying the recurrence to a power function. This gives the weak estimate `z_n = n^{alpha+o(1)}`.

Next, integral bounds and induction upgrade this to `z_n = Theta(n^alpha)`. This says that the exponent is correct up to constant factors.

The hard part is the final convergence of `z_n/n^alpha`. A natural idea would be to remove the maximum by proving monotonicity. If the sequence is increasing, then in each symmetric pair the larger index gives the larger value. But the initial values are `z_1 = 1` and `z_2 = 0`, so the sequence can fluctuate for a long time, especially near difficult parameter regimes.

The proof therefore proves eventual monotonicity. First, the consecutive differences vanish: `|Delta z_n| -> 0`. Since `z_n` is unbounded, the sequence eventually exceeds every fixed early value. A variation estimate then controls the remaining oscillations.

Once monotonicity holds, the max term can be replaced for all large `n`, and the recurrence becomes the delayed recurrence displayed on the slide. The last step is a dyadic argument proving that the series of absolute differences of `z_n/n^alpha` is summable. Therefore `z_n/n^alpha` converges to a positive constant, giving the exact asymptotic result.

## 10:45-11:45 | Back to the Stochastic Recurrence

Now we return to the original stochastic recurrence. Simulations suggest that the expectation `y_n` also grows like a power, say `C n^{alpha star}`. But the exponent appears to be larger than the deterministic lower-bound exponent.

The reason is that the lower bound replaced `E[max(X,Y)]` by `max(E[X],E[Y])`. In genuinely random cases, this inequality is strict. The maximum benefits from fluctuations: even when the two variables have the same mean, the larger of the two tends to be above that mean. So the stochastic recurrence has richer behavior than the deterministic proxy.

## 11:45-13:10 | A Fixed-Point Route

A natural way to study the stochastic recurrence is to normalize it. Suppose that `Y_n/n^alpha` converges in distribution. Then the limiting random variable must satisfy the fixed-point equation on the slide. The two factors `U^alpha` and `(1-U)^alpha` come from the asymptotic sizes of the two subtrees after normalization.

The standard approach would be a contraction method, but there are two obstructions. First, the fixed-point equation always has the degenerate solution `delta_0`. Second, `T_alpha` does not preserve the mean, so metrics requiring a fixed first moment are hard to use directly.

The workaround is to let the exponent move together with the distribution. For each distribution `mu` with mean one, choose `alpha(mu)` so that applying `T_alpha` preserves the mean. Then define the mean-preserving iteration `mu_{n+1} = T_{alpha_n}(mu_n)`. This is the candidate route toward the true stochastic exponent.

## 13:10-14:30 | Existence via Tychonoff

The fixed-point existence result uses Tychonoff's fixed-point theorem. The operator is `Phi(mu) = T_{alpha(mu)}(mu)`. The goal is to find a compact, convex set that is mapped into itself.

The set used in the memoir is `D_2(1,M)`: probability measures on the nonnegative real line with mean one and second moment bounded by `M`. Under the condition `u > 1/2`, one can choose `M` large enough so that `Phi` maps this set into itself.

The remaining ingredients are topological. In the space of bounded Radon measures with the weak topology, this set is nonempty, compact, and convex. Compactness comes from tightness, and the second-moment bound gives uniform integrability, which helps prove weak continuity of `Phi`.

Therefore Tychonoff gives a fixed point `mu star`, with `alpha star = alpha(mu star)`. This proves existence, but it is nonconstructive. It does not yet prove uniqueness, and it does not prove convergence of the mean-preserving iteration.

## 14:30-15:00 | Conclusion

To summarize, Hyperscan motivates mathematical questions because its performance depends on extracted literals and verification. Under a BST-like model, the longest-literal statistic leads to a stochastic recurrence with sums, maxima, and resets. We proved exact asymptotics for a deterministic lower bound, and the stochastic recurrence suggests a richer fixed-point problem.

The main open directions are to prove uniqueness of the fixed point, characterize the true exponent, and prove convergence of the mean-preserving iteration.
