# Randomness

## Entropy

Entropy is the measure of uncertainty, or disorder in a system.
The higher the entropy, the less the certainty found in the result.

We can compute the entropy of a probability distribution.

$$-p_1 \times log(p_1) - p_2 \times log(p_2) - \dots - p_N \times log(p_N)$$.

Entropy is maximized when the distribution is uniform because a uniform distribution maximizes uncertainty. Therefore, $$n$$-bit value can't have
more than $$n$$ bits of entropy.

## Random Number Generators (RNGs) and Pseudorandom Number Generators (PRNGs)