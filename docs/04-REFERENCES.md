# References & further reading

Primary disclosure and reporting on Randstorm. Figures and dates in this
repository are drawn from these sources.

- **Unciphered — "Randstorm: You Can't Patch a House of Cards"** (primary
  disclosure, November 2023).
  https://www.unciphered.com/disclosure-of-vulnerable-bitcoin-wallet-library-2/
- **The Hacker News — "Randstorm Exploit: Bitcoin Wallets Created b/w
  2011–2015 Vulnerable to Hacking"** (Nov 2023).
  https://thehackernews.com/2023/11/randstorm-exploit-bitcoin-wallets.html
- **Dark Reading — "'Randstorm' Bug: Millions of Crypto Wallets Open to
  Theft."**
  https://www.darkreading.com/application-security/randstorm-bug-millions-of-crypto-wallets-open-to-theft
- **Unchained — "$1 Billion in Old Bitcoin Wallets Vulnerable to Exploits:
  Report."**
  https://unchainedcrypto.com/1-billion-in-old-bitcoin-wallets-vulnerable-to-exploits-report/
- **TechTarget — "Cryptocurrency wallets might be vulnerable to 'Randstorm'
  flaw."**
  https://www.techtarget.com/searchsecurity/news/366559456/Cryptocurrency-wallets-might-be-vulnerable-to-Randstorm-flaw
- **keybleed.com** — Unciphered's public affected-address checker (enter only a
  public address).

## Background concepts

- BitcoinJS — the JavaScript Bitcoin library at the centre of the flaw.
- JSBN — the big-integer library whose `SecureRandom()` was the weak routine.
- `window.crypto.getRandomValues` — the correct browser CSPRNG the code should
  have used.
- Linear-congruential generators — the class of weak PRNG that browser
  `Math.random()` implementations of the era typically used.

## Note on figures

The **~1.4 million BTC** exposure figure and the billion-dollar
directly-at-risk framing come from Unciphered's disclosure and contemporaneous
reporting. The exploitable fraction varies by wallet creation date and software
version; these are ecosystem-scale estimates, not a claim about any specific
wallet.
