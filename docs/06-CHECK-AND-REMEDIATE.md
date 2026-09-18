# 6 · Are you affected? Check and remediate

For protecting **your own** wallets.

## 6.1 Am I in the affected population?

Likely at risk if **all** hold:

- created **2011–2015** (earlier = higher risk — see the CSPRNG timeline, ch. 5);
- created **in a web browser** (a web wallet or browser-based generator), not a
  desktop/mobile app or hardware wallet;
- the tool was built on **BitcoinJS / JSBN** of that era. Confirmed to include
  **Blockchain.info** (now Blockchain.com) web wallets; also many "paper wallet
  generator" sites and other browser wallets that reused the same `SecureRandom`.

Almost certainly **safe**: modern wallet software, hardware wallets (Ledger,
Trezor, Coldcard…), or anything post-2016 using the OS CSPRNG.

## 6.2 Check safely

- Unciphered published a checker at **keybleed.com** that tests whether an
  address is in the affected set.
- **Only ever enter a public address.** Never paste a private key or seed phrase
  into any website — a legitimate check needs only your public address; anyone
  asking for the secret is phishing.

## 6.3 Move the funds (the only real fix)

The weakness is frozen into the key; you cannot patch it. Stop using the key:

1. Create a **new wallet** with current, reputable software — ideally a
   **hardware wallet**.
2. Back up its seed and **test a small receive** first.
3. **Send the entire balance** from the old wallet to the new one.
4. Never reuse the old wallet/address.

Do it **now**. An at-risk key does not improve with age, and the balance is
public on-chain.

## 6.4 Don't

- Don't "wait and see" — the exposure is permanent.
- Don't move funds to another *old* wallet — it may share the flaw.
- Don't trust a "recovery service" that asks for your keys. If the wallet is
  yours and spendable, you move the funds yourself.
