# For developers: generating keys correctly

Randstorm is a cautionary tale about **where entropy comes from**. The lessons
generalize to any software that generates keys, nonces, tokens, or IVs.

## Rules

1. **Use the platform CSPRNG. Always.**
   - Browser: `crypto.getRandomValues(new Uint8Array(32))`.
   - Node.js: `crypto.randomBytes(32)`.
   - Python: `secrets` / `os.urandom`.
   - Never `Math.random()`, never a language's default `rand()`, never a
     time-seeded PRNG for anything security-bearing.

2. **Fail closed, not open.** If a secure source is unavailable, **stop with an
   error**. Randstorm happened partly because the code *fell back* to a weak
   source instead of refusing to generate a key. A wallet that won't generate a
   key is a minor annoyance; a wallet that generates a guessable one is a
   catastrophe.

3. **Don't roll your own RNG or your own feature-detection around it.** The
   original bug was a comparison/type error in detecting `window.crypto`. Prefer
   a single, well-tested primitive and let it throw if the environment can't
   support it.

4. **Entropy is not additive theatre.** Sprinkling a timestamp or mouse-move
   bits on top of a weak base does not make it strong. Bits of real,
   unpredictable entropy are what count — measure the *worst case*, not the
   average.

5. **Test the failure paths.** Unit-test what happens when the CSPRNG is
   missing. The vulnerable path was the one nobody exercised.

## Anti-patterns to grep your codebase for

- `Math.random()` anywhere near key/nonce/token/salt/IV generation.
- Custom `SecureRandom`-style shims that predate ubiquitous
  `getRandomValues`.
- Seeding a PRNG from `Date.now()` / `new Date()` / a timer.
- A `try/catch` around secure RNG that silently continues on failure.

## Further principle

The strength of every downstream secret is capped by the entropy of the source
that seeded it. No amount of good cryptography after the fact recovers entropy
that was never there.
