# 5 · CSPRNG availability timeline

Randstorm's severity gradient across 2011–2015 is largely explained by **when a
cryptographically secure RNG became available** in each browser. Before
`crypto.getRandomValues` existed, even correct wallet code had nothing strong to
draw from; the "fallback" *was* the primary path.

## 5.1 When `crypto.getRandomValues` shipped

| Browser | First version with `getRandomValues` | Approx. date |
|---|---|---|
| Chrome | 11 | **2011** |
| Safari (macOS) | 6.1 | 2012–2013 |
| iOS Safari | 7 | 2013 |
| Firefox | 21 | **2013** |
| Internet Explorer | 11 | 2013 |
| Opera | 15 | 2013 |
| Edge | 12 | 2015 |

(Availability per caniuse; see references.)

## 5.2 The exposure gradient

Reading §5.1 against the 2011–2015 window:

- **2011–2012:** essentially **only Chrome** had a CSPRNG. A wallet generated in
  Firefox, Safari, IE, or Opera in this period had **no secure source at all** —
  the weak `Math.random()`/timer path was unavoidable. These are the **weakest**
  wallets, consistent with the disclosure's "pre-March-2012 easiest" note.
- **2013:** Firefox, Safari, IE, and Opera all gain `getRandomValues`. From here,
  *correctly written* wallet code could get real entropy — but the BitcoinJS
  defect (§4.2) could still bypass it, and users on older browser builds remained
  exposed.
- **2014–2015:** CSPRNGs are widespread; BitcoinJS drops JSBN (March 2014);
  wallet code improves. Exposure **tapers** but does not vanish, because old
  browsers persist and old code lingers.

## 5.3 Why "which browser, which year" is the whole question

Combine this timeline with chapters 2–3:

$$H_{\text{eff}} = \min\bigl(256,\ H_{\text{state}}(\text{browser}),\ H_{\text{seed}}(\text{time window})\bigr),$$

but **only if the CSPRNG path failed.** If a real CSPRNG seeded the pool,
$H_{\text{eff}} = 256$ and the wallet is safe. So a wallet's fate turns on a
three-way question: *which browser* (state size), *what year* (was a CSPRNG even
available, and did the code use it), and *how tightly can its creation time be
bounded* (seed size). That is why no single "Randstorm number" applies to every
wallet — it is a family of exposures, tabulated above.
