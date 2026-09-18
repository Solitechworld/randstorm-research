# 4 · JSBN `SecureRandom` and the BitcoinJS defect

This chapter explains the code path that produced the keys — conceptually, to
show *where* the entropy was lost. It deliberately omits the byte-level mapping
from PRNG output to key and any reconstruction steps.

## 4.1 The intended design

BitcoinJS obtained key bytes from **`SecureRandom()`**, carried over from the
**JSBN** JavaScript big-integer library. Its design was reasonable on paper:

1. Maintain an **ARC4 (RC4) keystream pool** of 256 bytes as the byte source.
2. **Seed** that pool from the best available entropy:
   - if the browser exposes a CSPRNG (`window.crypto.getRandomValues`), seed
     from it;
   - always mix in a **timestamp** (`new Date().getTime()` / a timer) and other
     cheap values as *supplementary* entropy.
3. Draw key bytes from the seeded ARC4 stream.

If step 2 seeds ARC4 from a real CSPRNG, the output is fine. The whole security
rests on that "if."

## 4.2 Where it broke

Three failures, any one of which is sufficient, combined across the user base:

1. **A comparison / type defect** in the feature-detection meant the branch that
   should have pulled bytes from `window.crypto` **was not exercised as
   intended.** The strong seed the authors believed they were mixing in did not
   reliably reach the ARC4 pool.
2. **No CSPRNG existed** in many early-window browsers at all (chapter 5): before
   2013, most non-Chrome browsers had no `getRandomValues`. There was simply no
   strong source to read.
3. **The fallback was `Math.random()` + a single timer read.** When the strong
   path was absent or bypassed, ARC4 was effectively seeded from a
   non-cryptographic PRNG (chapter 3) plus a low-resolution clock.

## 4.3 Why the fallback is fatal (not merely weak)

Seeding a strong-looking ARC4 stream from a weak source does **not** launder the
weakness. ARC4 is deterministic: its entire output is a function of its seed. If
the seed carries only $H$ bits of real entropy, the key carries at most $H$ bits,
no matter how long or random the ARC4 stream *looks*. This is the ceiling theorem
of §2.1 applied one layer down:

$$H_\infty(\text{key}) \le H_\infty(\text{ARC4 seed}) \le H_\infty(\texttt{Math.random state} \;\Vert\; \texttt{timer}).$$

A single medium-resolution timer contributes only a handful of unknown bits
around a knowable moment — it cannot rescue a 48-bit (or smaller) base. The
disclosure's own phrasing is that effective entropy was **"substantially less
than 48 bits"** in some configurations.

## 4.4 The end of the window

BitcoinJS **discontinued JSBN in March 2014**. Wallet software then migrated to
better sourcing over the following ~18 months, which is why the exposure tapers
through 2014–2015 rather than ending sharply. A wallet's real exposure depends on
the *exact* library version and browser it was created with.
