# 7 · For developers: generating keys correctly

Randstorm is a lesson about **where entropy comes from**. It generalizes to any
key, nonce, token, salt, or IV.

## 7.1 Rules

1. **Use the platform CSPRNG. Always.**
   - Browser: `crypto.getRandomValues(new Uint8Array(32))`
   - Node.js: `crypto.randomBytes(32)`
   - Python: `secrets` / `os.urandom`
   - Never `Math.random()`; never a default `rand()`; never a time-seeded PRNG
     for anything security-bearing.
2. **Fail closed.** If a secure source is unavailable, **throw** — never fall
   back to a weak one. Randstorm happened because the code fell back instead of
   refusing. A wallet that won't generate a key is a nuisance; one that generates
   a guessable key is a disaster.
3. **Don't hand-roll RNG feature-detection.** The original defect was a
   comparison/type error while detecting `window.crypto`. Use one well-tested
   primitive and let it throw.
4. **Entropy is not additive theatre.** A timestamp or mouse-move sprinkled on a
   weak base does not make it strong. Count the **worst-case (min-)entropy**, not
   the average.
5. **Test the failure paths.** Unit-test "CSPRNG missing." The vulnerable branch
   was the one nobody exercised.

## 7.2 Grep your codebase for

- `Math.random()` near key / nonce / token / salt / IV.
- Custom `SecureRandom`-style shims predating ubiquitous `getRandomValues`.
- Seeding a PRNG from `Date.now()` / `new Date()` / a timer.
- `try/catch` around secure RNG that silently continues on failure.

## 7.3 The principle

Every downstream secret is capped by the entropy of the source that seeded it.
No cryptography after the fact recovers entropy that was never there.
