# Delivery Notes

Audience model:

- One evaluator is your supervisor, so they already know the technical path. Use them as the precision anchor: state hypotheses, what is proved, and what remains open.
- Laurent Hauswirth works in geometry and curvature, with public work around minimal surfaces, constant-mean-curvature surfaces, geometric analysis, and architectural geometry. For him, the best bridge is not Hyperscan internals; it is the mathematical structure: recursive objects, asymptotic scaling, compactness, continuity, and fixed-point existence.

## Main Strategy

Deliver the talk as a mathematical extraction story:

> A complicated engine motivates a clean random object; the random object gives a recurrence; the recurrence has a deterministic tractable shadow; the original stochastic problem suggests a fixed-point problem.

This keeps the systems motivation useful without forcing a geometry audience to absorb implementation details.

## What To Emphasize

1. In the first two slides, do not over-explain Hyperscan.
   Say only that Hyperscan extracts literals, uses them as fast SIMD filters, and verifies the remaining automata afterward. The key sentence is: longer literals are more selective, so their length is a meaningful statistic.

2. On the random model slide, speak slowly.
   This is where the non-CS audience learns the object. Say: "This is just a probability distribution on syntax trees." Then explain the three root cases: concatenation, alternation, Kleene star.

3. On the recurrence slide, translate each term before reading the formula.
   Concatenation gives a sum. Alternation gives a max. Kleene star gives zero. Then the formula becomes natural.

4. On the deterministic theorem slide, separate heuristic from theorem.
   Say: "The equation for alpha comes from the ansatz, but the theorem proves the ansatz is correct for the lower-bound recurrence." This avoids sounding like the proof is just experimental fitting.

5. On the deterministic proof skeleton, do not walk through every lemma equally.
   Spend most time on the obstacle: the max term cannot be removed until eventual monotonicity is proved. The audience only needs the shape:
   Riemann sum -> rough power -> induction bounds -> eventual monotonicity -> delayed recurrence -> convergence of the normalized sequence.

6. On the stochastic fixed-point slides, lean into the geometry/analysis language.
   Present Tychonoff as a direct-method style argument:
   a priori second-moment bound -> compact convex domain -> continuity -> existence.
   Make clear that this is existence only, not uniqueness or convergence.

## Good Phrases To Use

- "I am not trying to model every implementation detail; I am isolating one statistic that the implementation naturally suggests."
- "The recurrence has the same flavor as many self-similar limit problems: the large object is built from rescaled independent copies."
- "The deterministic recurrence is a tractable shadow of the stochastic recurrence."
- "The fixed-point result is nonconstructive. It gives existence, but not yet the dynamics of the iteration."
- "The open problem is to return from this fixed-point picture to the asymptotics of the original expectation."

## What To Avoid

- Do not spend more than two minutes on Hyperscan architecture.
- Do not present the Snort fit as proof that the model is realistic; present it only as evidence that the parameters can be calibrated.
- Do not say the stochastic exponent is proved. Say simulations suggest it.
- Do not describe Tychonoff as if it proves convergence of the iteration.
- Do not read every symbol in the proof skeleton. Use the diagram to guide the story.

## If Questions Come

If asked why this is interesting for geometry:

> The common point is not the object itself, but the method: we look for a limiting object after rescaling, prove a priori bounds, use compactness, and then identify a fixed point.

If asked why the deterministic lower bound matters:

> It is the first recurrence where the max survives. It is simpler than the stochastic recurrence but still contains the main technical obstruction.

If asked what is missing:

> The missing steps are uniqueness of the fixed-point distribution, characterization of the true exponent, and convergence of the mean-preserving iteration or of the normalized stochastic recurrence.
