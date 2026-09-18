# 8 · References & further reading

## Primary disclosure & reporting

- **Unciphered — "Randstorm: You Can't Patch a House of Cards"** (primary
  disclosure, Nov 2023).
  https://www.unciphered.com/disclosure-of-vulnerable-bitcoin-wallet-library-2/
- **The Hacker News — "Randstorm Exploit: Bitcoin Wallets Created b/w
  2011–2015 Vulnerable to Hacking"** (Nov 2023).
  https://thehackernews.com/2023/11/randstorm-exploit-bitcoin-wallets.html
- **Dark Reading — "'Randstorm' Bug: Millions of Crypto Wallets Open to Theft."**
  https://www.darkreading.com/application-security/randstorm-bug-millions-of-crypto-wallets-open-to-theft
- **Unchained — "$1 Billion in Old Bitcoin Wallets Vulnerable to Exploits."**
  https://unchainedcrypto.com/1-billion-in-old-bitcoin-wallets-vulnerable-to-exploits-report/
- **TechTarget — "Cryptocurrency wallets might be vulnerable to 'Randstorm'."**
  https://www.techtarget.com/searchsecurity/news/366559456/Cryptocurrency-wallets-might-be-vulnerable-to-Randstorm-flaw
- **keybleed.com** — Unciphered's affected-address checker (public address only).

## Browser RNG internals

- **V8 — "There's Math.random(), and then there's Math.random()"** (Dec 2015):
  MWC1616 → xorshift128+, 128-bit state, period $2^{128}-1$; Chrome 49.
  https://v8.dev/blog/math-random
- **LWN — "Randomness in the web browser"** (Dec 2015): survey of SpiderMonkey
  (Java LCG), V8 (MWC1616 concatenation flaw), and JavaScriptCore (GameRand),
  with TestU01 results.
  https://lwn.net/Articles/666407/
- **Mozilla bug 322529** — "Upgrade Math.random() to XorShift128+."
  https://bugzilla.mozilla.org/show_bug.cgi?id=322529
- **Hackaday — "V8 JavaScript Fixes (Horrible!) Random Number Generator"** (Dec
  2015). https://hackaday.com/2015/12/28/v8-javascript-fixes-horrible-random-number-generator/

## CSPRNG availability

- **caniuse — `crypto.getRandomValues()`** (per-browser version support).
  https://caniuse.com/getrandomvalues
- **MDN — `Window.crypto` / `Crypto.getRandomValues()`.**
  https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues

## Background

- `java.util.Random` LCG constants ($a=$ `0x5DEECE66D`, $c=$ `0xB`, $m=2^{48}$) —
  the recurrence SpiderMonkey inherited.
- secp256k1 / ECDSA — the signature scheme whose keys are affected (the scheme
  itself is not broken; only the key *generation* was).

## Note on figures

The **~1.4 million BTC** exposure figure and the billion-dollar directly-at-risk
framing come from Unciphered's disclosure and contemporaneous reporting. The
exploitable fraction varies by creation date, browser, and software version;
these are ecosystem-scale estimates, not claims about any specific wallet.
