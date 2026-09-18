# 1 · Cryptographic background

## 1.1 What a Bitcoin key actually is

A Bitcoin private key is an integer $k$ in the range

$$1 \le k < n,\qquad n \approx 2^{256}$$

where $n$ is the order of the secp256k1 group. The public key is $K = kG$ (scalar
multiplication of the generator point $G$), and the address is a hash of $K$.
Everything—receiving, signing, spending—derives from $k$. There is no password,
no server, no recovery: **the key is the account.**

## 1.2 Why the key must be uniform

secp256k1 has no known structural weakness; the fastest attack on a *single*
key is generic (Pollard's rho), costing about $\sqrt{n} \approx 2^{128}$ group
operations. That is the security you are paying for. But it only holds if $k$ was
chosen **uniformly at random** from its range. If $k$ is instead drawn from a
much smaller set $S \subset [1, n)$ that an attacker can enumerate, the cost of
finding it drops from $2^{128}$ to roughly $|S|/2$ — and $|S|$, not $n$, is what
matters.

Randstorm is entirely about $|S|$ being small.

## 1.3 Entropy: the right measure

Two definitions matter, and they are **not** the same.

- **Shannon entropy** $H = -\sum_i p_i \log_2 p_i$ measures *average*
  unpredictability. It is the wrong tool for key security, because an attacker
  guesses the *most likely* candidates first.
- **Min-entropy** $H_\infty = -\log_2\bigl(\max_i p_i\bigr)$ measures the
  *worst case* — the probability of the single most likely value. This is the
  correct measure for keys: it bounds how fast an optimal attacker succeeds.

For a source that is uniform over $N$ values, $H_\infty = H = \log_2 N$. For a
skewed source (e.g. a time-seeded PRNG where recent timestamps are more likely),
$H_\infty < H$, and the key is **weaker than the average would suggest.**

Throughout this report, "entropy" means **min-entropy** unless stated otherwise,
because that is what an attacker's workload depends on.

## 1.4 The generator is the whole game

A key is produced by:

$$\text{seed } s \;\xrightarrow{\ \text{PRNG}\ }\; \text{bytes} \;\xrightarrow{\ \text{reduce mod } n\ }\; k.$$

If the PRNG is deterministic (all non-cryptographic PRNGs are), then $k$ is a
pure function of $s$. The randomness of $k$ is *exactly* the randomness of $s$
plus whatever additional entropy is folded in during generation — never more.
This is the inequality the next chapter formalizes.
