# Are you affected? How to check and move funds

This guide is for protecting **your own** wallets.

## Am I in the affected population?

You may be affected if **all** of these are true:

- The wallet was created **between 2011 and 2015** (earlier = higher risk).
- It was created **in a web browser** — a web wallet or a browser-based
  generator — rather than by a desktop/mobile app or a hardware wallet.
- The tool was built on **BitcoinJS / JSBN** of that era. Notable example:
  **Blockchain.info** (now Blockchain.com) web wallets from that period. Many
  other browser wallets and "paper wallet generator" sites of the time reused
  the same `SecureRandom` code and inherit the flaw.

You are almost certainly **not** affected if the wallet was made by modern
wallet software, a hardware wallet (Ledger, Trezor, Coldcard, …), or any tool
released after ~2016 that uses the operating-system CSPRNG.

## How to check safely

- **Unciphered published a checker** at **keybleed.com** that tests whether an
  address is in the affected set. Treat any such site with care: only ever
  enter a **public address**, never a private key or seed phrase.
- **Never paste a private key or seed phrase into any website** to "check" it.
  A legitimate check only needs your public address. Anyone asking for the
  secret is phishing.

## If you might be affected — move the funds

Because the weakness is frozen into the key, **the only fix is to stop using
that key**:

1. Create a **new wallet** with current, reputable software — ideally a
   **hardware wallet**, or a well-reviewed modern mobile/desktop wallet.
2. **Verify you control the new wallet** (back up its seed, test a small
   receive first).
3. **Send the entire balance** from the old wallet to the new one.
4. Do not reuse the old wallet or address again.

Do this **sooner rather than later**. An at-risk key does not get safer with
time, and the balance is visible on-chain to anyone watching.

## What not to do

- Don't "wait and see." The exposure is permanent.
- Don't move funds to another *old* wallet — it may share the flaw.
- Don't trust a "recovery service" that asks for your keys. If a wallet is
  yours and still spendable, you move the funds yourself with the steps above.
