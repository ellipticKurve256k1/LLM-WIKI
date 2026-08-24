# Bitcoin Chainwork

A valid Bitcoin block must have a hash below the target:

`hash ≤ target`

The probability of finding a valid hash in one attempt is approximately:

`p = target / 2^256`

The **reciprocal** of a number is `1/x`.
- reciprocal: number that makes 1 when multiplying

For example:

`p = 1/100`

Its reciprocal is:

`1/p = 100`

So if the probability of success is `p`, the expected number of attempts for one success is:

`Expected Work = 1/p`

For Bitcoin:

`Expected Work ≈ 2^256 / target`

`Chainwork` is the sum of the expected work of all blocks from genesis to the current tip:

`Chainwork = Σ Expected Work`

In short:

`Target → Success Probability → Reciprocal → Expected Work → Chainwork`

> Expected work does not mean a block is guaranteed to be found after that many hashes. It is a statistical expectation.

## Why the Reciprocal Represents Expected Work

Let:

- `p` = probability of success for one hash
- `N` = number of hash attempts

The expected number of successes after `N` attempts is:

`Expected successes = N × p`

If we want the expected number of successes to be `1`:

`N × p = 1`

So:

`N = 1 / p`

This is why the reciprocal of the success probability gives the **expected number of attempts for one success**.

For Bitcoin:

`p ≈ target / 2^256`

Therefore:

`Expected Work ≈ 2^256 / target`

Example:

`p = 1/100`

Then:

`100 × 1/100 = 1`

So 100 attempts give an expected value of one success.

This does **not** guarantee one success after exactly 100 attempts. It is a statistical expectation.