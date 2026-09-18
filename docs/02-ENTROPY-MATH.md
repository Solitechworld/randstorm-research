# 2 · The entropy mathematics

This chapter derives, with numbers, why Randstorm keys are searchable. It
contains **no reconstruction algorithm** — only the accounting that bounds the
size of the search space.

## 2.1 The ceiling theorem

Let a key be generated deterministically from a seed/state $s$:

$$k = f(s), \qquad s \in S.$$

Then:

$$\bigl|\{\,k : s \in S\,\}\bigr| \le |S| \quad\Longrightarrow\quad H_\infty(k) \le \log_2 |S|.$$

If several independent entropy inputs $s_1,\dots,s_m$ are combined, the ceiling
is their **sum in bits**, capped at the key length:

$$H_{\text{eff}} = \min\!\Bigl(256,\ \sum_{j=1}^{m} \log_2 |S_j|\Bigr).$$

Two inputs dominate in Randstorm: the PRNG's **internal state** and the PRNG's
**seed**. Because the state is itself produced from the seed, the binding
constraint is usually the **smaller** of the two:

$$H_{\text{eff}} = \min\bigl(256,\ H_{\text{state}},\ H_{\text{seed}}\bigr).$$

## 2.2 Expected attacker work

For a target drawn uniformly from $N = 2^{b}$ candidates, the expected number of
trials to find it (searching in any fixed order) is

$$\mathbb{E}[\text{trials}] = \frac{N+1}{2} \approx 2^{\,b-1}.$$

So an entropy of $b$ bits costs about $2^{b-1}$ key derivations + address checks.
The table in §2.5 turns that into wall-clock intuition.

## 2.3 The seed is usually a clock

Non-cryptographic PRNGs must be seeded. In a browser with no CSPRNG available,
the only readily available "unpredictable" quantity is **time**. A generator
seeded from a millisecond timestamp has a seed space equal to the number of
milliseconds in the window an attacker must consider:

$$N_{\text{seed}} = \Delta t_{\text{ms}}, \qquad H_{\text{seed}} = \log_2 \Delta t_{\text{ms}}.$$

| Window an attacker must cover | Milliseconds $\Delta t$ | $H_{\text{seed}} = \log_2 \Delta t$ |
|---|---:|---:|
| 1 hour | $3.6\times10^{6}$ | **21.8 bits** |
| 1 day | $8.64\times10^{7}$ | **26.4 bits** |
| 1 month | $2.63\times10^{9}$ | **31.3 bits** |
| 1 year | $3.156\times10^{10}$ | **34.9 bits** |
| 5 years (2011–2015) | $1.58\times10^{11}$ | **37.2 bits** |

The moment an attacker can bound *when* a wallet was made — and wallet creation
often leaves a public first-funding timestamp on-chain — the seed space is one of
these rows, not $2^{256}$.

## 2.4 Composing the bounds

Effective entropy is the minimum of the state ceiling and the seed ceiling:

- **Firefox / IE (48-bit LCG)**, seed from time-to-the-day:
  $H_{\text{eff}} = \min(256,\ 48,\ 26.4) = \mathbf{26.4\ bits}.$
- **Firefox / IE (48-bit LCG)**, seed unknown within the whole 5-year window:
  $H_{\text{eff}} = \min(256,\ 48,\ 37.2) = \mathbf{37.2\ bits}.$
- **V8 / Chrome (MWC1616, $2^{32}$ distinct outputs)**, seed within a year:
  $H_{\text{eff}} = \min(256,\ 32,\ 34.9) = \mathbf{32\ bits}$ (state-bound).
- **V8 with a badly-chosen state** (V8's own note: cycle can fall below
  40 million $\approx 2^{25.3}$): $H_{\text{eff}} \le \mathbf{25.3\ bits}.$

## 2.5 What those bit-counts mean in time

Purely to convey **severity** (this is a feasibility bound, not a procedure), at
an illustrative rate of $R$ candidate keys derived-and-checked per second:

$$T \approx \frac{2^{\,H_{\text{eff}}-1}}{R}.$$

| $H_{\text{eff}}$ | Candidates $2^{H-1}$ | $T$ at $R=10^{6}/s$ | $T$ at $R=10^{9}/s$ |
|---:|---:|---:|---:|
| 25.3 | $\approx 2.6\times10^{7}$ | ~26 s | ~0.03 s |
| 26.4 | $\approx 5.5\times10^{7}$ | ~55 s | ~0.06 s |
| 32 | $\approx 2.1\times10^{9}$ | ~36 min | ~2.1 s |
| 37.2 | $\approx 7.9\times10^{10}$ | ~22 h | ~79 s |
| 48 | $\approx 1.4\times10^{14}$ | ~4.5 yr | ~1.6 days |
| 128 (rho on one good key) | $2^{127}$ | — | infeasible |
| 256 (ideal brute force) | $2^{255}$ | — | infeasible |

The jump from the bottom rows to the top rows **is** Randstorm. A properly
generated key sits at 128+; a worst-case 2011 wallet sits near 25.

## 2.6 Why the reduction is $2^{200+}$

From the ideal $2^{256}$ to a time-seeded 48-bit LCG bounded to a day
($2^{26.4}$), the search space shrinks by a factor of

$$2^{256}/2^{26.4} = 2^{229.6}.$$

No cryptography downstream can restore that. The entropy was never present to
begin with; ECDSA faithfully preserves the little that was.
