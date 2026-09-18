# The weakness in depth

This is a conceptual explanation of *why* Randstorm keys are weak. It does not
include a keyspace reduction, seed enumerator, or any code that would recover a
key — only the reasoning that shows the entropy was insufficient.

## Where the randomness came from

A Bitcoin private key is a 256-bit integer that must be drawn uniformly at
random. Browser wallets built on **BitcoinJS** obtained that randomness through
the **`SecureRandom()`** routine carried over from the **JSBN** big-integer
library. `SecureRandom` was supposed to seed an ARC4-based stream from a strong
platform source and read key bytes out of it.

The intended strong source was the browser CSPRNG —
`window.crypto.getRandomValues` (and its earlier vendor-prefixed forms). When
that is available and used correctly, the output is fine.

## Why the strong path was not taken

Three things combined so that, for a large population of users, the secure seed
was never actually used:

1. **A comparison / type error.** A bug in the feature-detection meant the code
   path that would have pulled bytes from `window.crypto` was not exercised as
   intended, so the CSPRNG seed did not make it into the pool the way the authors
   believed it did.
2. **Browsers that had no CSPRNG yet.** In the early part of the window
   (2011–2013), many shipping browsers did **not** expose
   `window.crypto.getRandomValues` at all. There was simply nothing strong to
   read from.
3. **A weak fallback.** When the strong source was missing (or bypassed by the
   bug), the pool was seeded from **`Math.random()`** plus a single
   medium-resolution timer reading.

## Why the fallback is fatal

`Math.random()` is not a cryptographic RNG. In the browsers of that era it was
typically a **linear-congruential generator with a 48-bit internal state**. Two
consequences follow, and either one is disqualifying:

- **The reachable keyspace is bounded by the seed, not the key.** A 256-bit key
  produced from a generator whose state is 48 bits can only take at most 2⁴⁸
  distinct values, not 2²⁵⁶. That is an astronomically smaller haystack.
- **LCG state is recoverable and the seed space is small.** LCGs are not
  one-way; their state is often further constrained by how the browser seeded it
  (frequently from wall-clock time at page load), shrinking the practical search
  far below even 48 bits.

Adding a single medium-resolution timer reading does not rescue this: a
low-resolution timestamp around a known moment contributes only a handful of
unknown bits, not the ~200 the key is short by.

The disclosure's own wording is that the effective entropy was **"substantially
less than 48 bits"** in some configurations. The exact figure depends on the
browser, its `Math.random()` implementation, and how the wallet software drove
`SecureRandom`.

## Why later wallets are harder

The exposure is not uniform across 2011–2015:

- **Pre-March-2012** wallets are the weakest — the smallest effective seed
  spaces and the least CSPRNG availability.
- Over time, browsers shipped `window.crypto.getRandomValues`, some wallet code
  paths improved, and the ecosystem migrated away from JSBN (**BitcoinJS dropped
  JSBN in March 2014**). Wallets from **2014–2015** are therefore
  **substantially more difficult** to attack, though not automatically safe —
  a wallet's real exposure depends on the specific software and browser that
  made it.

## The takeaway

The security of a key is the security of the process that generated it. A key
that "looks" like 256 random bits but came from a 48-bit (or smaller) seed has
the strength of that seed. That is the whole of Randstorm: not a break in ECDSA
or in Bitcoin, but a broken *source of randomness* upstream of the key.
