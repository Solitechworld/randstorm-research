# 3 · Browser RNG internals (2011–2015)

Randstorm's severity is per-browser, because each engine shipped a *different*
weak `Math.random()`. This chapter documents each one: the algorithm, its state
size, why it fails, and when it was fixed. All engines listed here were replaced
by **xorshift128+** (128-bit state, period $2^{128}-1$) in late 2015 / 2016 —
after the vulnerable window.

## 3.1 V8 — Chrome, Opera, Android WebView (MWC1616)

- **Algorithm:** MWC1616 — two 16-bit *multiply-with-carry* sub-generators, one
  for the high half and one for the low half of the output, concatenated.
- **State:** 64 bits nominal (two 32-bit words), but the design yields at most
  **$2^{32}$ distinct output values** — far short of the $2^{52}$ doubles a
  float in $[0,1)$ can represent.
- **The concatenation defect:** V8 *concatenated* the two halves instead of
  mixing (XOR) them. This dropped the effective cycle dramatically — on 64-bit
  systems, from a nominal $2^{60}$ toward as little as $2^{15}$, producing
  visible collisions (~1 in 30,000 generated identifiers).
- **Bad-state cycles:** V8's own writeup notes that "with a badly chosen initial
  state, the cycle length could be less than 40 million" ($\approx 2^{25.3}$).
- **Fixed:** replaced with xorshift128+ in **V8 v4.9.41.0**, shipped in
  **Chrome 49** (announced Dec 2015).

**Bound it imposes:** $H_{\text{state}} \le 32$ bits, and materially less with an
unlucky state.

## 3.2 SpiderMonkey — Firefox (Java-derived 48-bit LCG)

- **Algorithm:** a linear congruential generator "imported from Java decades
  ago." This is the classic `java.util.Random` recurrence:

  $$X_{n+1} = (a X_n + c) \bmod 2^{48}, \qquad a = \texttt{0x5DEECE66D},\ c = \texttt{0xB}.$$

- **State:** **48 bits.** The *entire* output sequence a wallet consumes is a
  deterministic function of that 48-bit state — so at most $2^{48}$ distinct
  keystreams exist, before seeding is even considered.
- **Output precision defect:** it returned only ~32 bits of usable precision
  instead of the 53 bits the ECMAScript spec requires, and failed 12 of 96
  TestU01 *Crush* tests (and 1 of 10 *SmallCrush*).
- **Fixed:** replaced with xorshift128+ (Mozilla bug 322529), landing in stable
  Firefox in early 2016.

**Bound it imposes:** $H_{\text{state}} \le 48$ bits; with time-seeding, the
*seed* bound of §2.3 (≈22–35 bits) dominates.

## 3.3 JavaScriptCore — Safari (GameRand)

- **Algorithm:** **GameRand** — an extremely lightweight generator intended for
  games, not cryptography. Very short period, very little state.
- **Quality:** scored "similarly poor numbers" on TestU01 to the LCG engines.
- **Fixed:** replaced with xorshift128+ on **30 November 2015**.

**Bound it imposes:** small state; Safari-era wallets are among the weakest when
GameRand was in use.

## 3.4 Internet Explorer / early Edge

- **Algorithm:** reportedly the **same LCG family as Firefox** in IE 11; Edge's
  early behavior was not fully characterized (test harnesses crashed).
- **Bound:** treat as the 48-bit-LCG case.

## 3.5 Comparison

| Engine | Algorithm (2011–2015) | Nominal state | Effective distinct outputs / cycle | Fixed |
|---|---|---:|---|---|
| V8 (Chrome/Opera) | MWC1616 | 64-bit | $2^{32}$ outputs; cycle can fall < $2^{25.3}$ | Chrome 49, Dec 2015 |
| SpiderMonkey (Firefox) | Java 48-bit LCG | 48-bit | $\le 2^{48}$ keystreams; ~32-bit precision | Firefox ~early 2016 |
| JavaScriptCore (Safari) | GameRand | small | short period | 30 Nov 2015 |
| IE 11 | LCG (Firefox-like) | 48-bit | $\le 2^{48}$ | — |
| **All → replacement** | **xorshift128+** | **128-bit** | period $2^{128}-1$ | late 2015 / 2016 |

## 3.6 The crucial caveat

Even xorshift128+ **is not cryptographically secure** — V8, Mozilla and WebKit
all say so explicitly. The 2016 fixes made `Math.random()` *statistically* good,
not *cryptographically* safe. For keys, the only correct source is
`crypto.getRandomValues` (chapter 5). A wallet that used `Math.random()` was
unsafe before *and* after these fixes; the fixes simply removed the extra,
grosser weakness on top.
