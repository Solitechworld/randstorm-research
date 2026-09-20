# 9 · Search economics: what it actually takes to sweep the space

> **Read this first — deliberate redactions.** This chapter analyses the
> *economics* of a Randstorm keyspace sweep: where the compute goes, why some
> architectures win, and what the measured cost envelope looks like on modern
> GPUs. It is based on a working research implementation. **The operational
> core is intentionally withheld**, and we are explicit about what is missing:
> the byte-level state→key derivation, the candidate enumeration order, the
> match-filter design, the precomputed-table geometry, and the validation
> traces that pin a search to real wallet generators. Each omission is marked
> inline. This is responsible disclosure: the analysis is complete enough to
> replicate the *economics*; it is deliberately incomplete as a *capability*.
> See §9.6 for the full redaction ledger.

## 9.1 The shape of the problem

A Randstorm sweep is not cryptanalysis in the classical sense. No lattice, no
discrete-log shortcut, no collision search over the curve. It is the
**exhaustive evaluation of a pure function** over a bounded, enumerable domain:

$$
\text{candidates} \;=\; \{\,(s, t)\,\}, \qquad |S|\cdot|T| \;\text{pairs},
$$

where one axis is a small PRNG state space and the other is a time window.
Every pair evaluates independently to a key candidate; every key candidate is
checked against a target set. Three properties follow, and they drive every
design decision worth making:

1. **Embarrassingly parallel.** No communication between candidates. N GPUs
   divide the domain; nothing else is shared. Multi-GPU scaling is a
   partitioning problem, not an algorithmic one — measured implementations
   reach **~98% scaling on two devices**, and the limit is host I/O, not
   arithmetic.
2. **Bound by one primitive.** Per candidate, the dominant cost is a
   **fresh 256-bit elliptic-curve scalar multiplication** against a fixed base
   point. This single fact determines the entire cost model (§9.2).
3. **Window-linear.** The domain size is the product of the two axes, so the
   time axis is where the money is. Forensic work that narrows a wallet's
   creation window from a month to a day shrinks the sweep by ~30×; from a
   week to an hour, ~168×. **No GPU purchase beats a good time bound.**

**[Withheld]** the exact parametrisation of the two axes, the state-space
bound used in practice, and the enumeration order that guarantees
gap-free, duplicate-free coverage. Together these turn "the space exists"
into "here is a complete sweep schedule" — that is the line we do not cross
in public.

## 9.2 The anatomy of one candidate

Fix a modern GPU (RTX-4090 class) and cost one candidate in 32-bit integer
operations. The arithmetic is dominated by three stages:

| Stage | What it does | Order of cost | Notes |
|---|---|---|---|
| PRNG + pool expansion | small-state PRNG draws, byte assembly, a 256-byte key-schedule mixing pass | ~1% | trivial; table-friendly |
| **Fixed-base scalar mult.** | `k·G` via a precomputed-table method | **~70%** | ≈ 25,000 32-bit multiply-class ops; **the floor** |
| Hashing + match | two hash passes + address encoding + filter | ~10–20% | two variants of the pubkey encoding double the hashing tail |

The remaining large cost is the **field inversion** traditionally needed to
convert projective curve results back to affine coordinates for hashing. A
naïve per-candidate inversion (Fermat's little theorem: ~256 squarings) adds
roughly 20,000 further multiply-class operations — nearly doubling the bill.
The known fix is **Montgomery's trick**: batch N candidates, pay **one**
inversion for the whole batch, and recover each individual inverse with
prefix/suffix products. In measured A/B runs this is worth **~1.5×**
end-to-end — the single largest algorithmic lever available to this workload.

The instructive negative result: a **warp-cooperative** variant that
amortises the inversion across 32 lanes with shuffle-based product scans
*loses* to the simple per-thread batch on this workload (~0.9×). The scan's
~80 lockstep shuffle instructions per candidate cost more than the amortised
inverse saves, because each thread's dependency chain is already long and
independent. **On embarrassingly-parallel crypto, lockstep cooperation is
usually a net tax** — a lesson that generalises far beyond this problem.

**[Withheld]** the batch size, the exact table geometry of the
fixed-base multiplication, and the field-representation choices (limb width,
reduction strategy) that make the multiply chains fast. These are the
difference between a competent implementation and a competitive one.

## 9.3 The kangaroo fallacy: why "10 Gkeys/s" is a category error

Public discourse about Randstorm tooling regularly cites multi-Gkey/s
throughput. That figure belongs to **Pollard-kangaroo class tools**, whose
inner loop is a *single elliptic-curve point addition* per step (a few hundred
nanoseconds). A Randstorm sweep has no such structure: **every candidate
requires an independent full scalar multiplication**, three orders of
magnitude more arithmetic than a kangaroo hop.

The honest accounting:

$$
\text{ceiling} \;\approx\; \frac{\text{issue rate}}{\text{muls per candidate at } B{=}8}
\;\approx\; 700\ \text{Mkeys/s} \quad \text{(4090 class, multiply-bound)}
$$

— and that ceiling assumes perfect issue utilisation that no realistic kernel
reaches. **Measured** implementations, on three GPU classes across two
toolchains, land in the **25–83 Mkeys/s per device** envelope (39 Mkeys/s on
an RTX 3090; 83 Mkeys/s on a full-clock RTX 4090; ~163 Mkeys/s across two
4090s at 98% scaling). Occupancy studies confirm the kernel is
**compute-bound on per-candidate work**: register capping, occupancy floors,
and scheduling changes all fail to move the number. The gap between 10 Gkeys/s
and ~80 Mkeys/s is not engineering slack; it is **~121×, and it is
arithmetic**.

Practical consequence: at ~80 Mkeys/s, a one-week time window is
~1.9 × 10¹⁷ candidates ≈ **~27 GPU-days on a single 4090-class device** —
comfortable for a well-funded recovery engagement, hopeless for an ad-hoc
attacker with one gaming PC. The economics are the safety margin.

## 9.4 Windows, not wallets: where the leverage is

The cost model above makes one thing clear: the sweep's feasibility is set by
the **time-axis bound**, and the time axis is an *information* problem, not a
compute problem. Evidence that narrows creation time:

- wallet-file / backup filesystem timestamps,
- first-seen on-chain activity (a hard lower bound for funded addresses),
- exchange deposit records, email confirmations, block-explorer metadata,

each translate directly into GPU-months saved. This is why serious recovery
engagements begin with forensics, not hardware procurement — and why the
pricing of such engagements (§ README) is dominated by evidence quality
rather than GPU count.

A second, less obvious observation from implementation work: the
seed→key map is **not injective**. Distinct seeds collapse to identical
wallets (a planted test wallet was recovered from two different states). The
effective key population is therefore *smaller* than the nominal candidate
count — Randstorm is marginally *worse* than the entropy math already says.

## 9.5 Trust: why every hit is a claim, not a fact

A production-grade sweep cannot trust its own device. GPUs overclock,
underflow memory, and silently corrupt arithmetic; a single flipped bit in a
hash pass produces a plausible-looking false negative — or worse, a
false-positive "recovery". The architecture that survives audit separates
**detection** from **proof**:

- the GPU runs a cheap **statistical filter** (a truncated digest comparison)
  whose false-positive rate is calculable (≈ 2⁻⁶⁴ per candidate here);
- **every filter hit is re-derived on the CPU** against the full 20-byte
  target digest before it is reported;
- the device must pass known-answer tests against a **live-captured
  reference vector** before any search is permitted, and a device that fails
  mid-run is quarantined and its work redistributed;
- long runs checkpoint atomically (write-temporary → fsync → rename), so a
  crash costs seconds, never correctness.

**[Withheld]** the filter's construction, its exact digest
truncation, and the reference vectors themselves. A filter design plus the
vectors is most of a working verifier; the verifier is most of a working tool.

## 9.6 The redaction ledger

For public safety, the following components of a working sweep are **known to
us and deliberately unpublished**. Each is listed with the reason it matters.
This is the complete list — nothing else stands between this chapter and a
functioning tool, which is precisely why it stays private.

| # | Withheld component | Why it matters | Release policy |
|---|---|---|---|
| 1 | Byte-level PRNG→pool→key derivation (incl. the 33-byte extraction convention) | The single fact that separates real recoveries from infinite misses | Never public; licensed implementations only |
| 2 | Candidate enumeration & stripe encoding | Turns the space into a gap-free, resumable, multi-machine schedule | Never public |
| 3 | Match-filter construction + target encoding | The detection layer; defines the false-positive economics | Never public |
| 4 | Fixed-base table geometry & field representation | The 2–3× implementation competitive edge | Never public |
| 5 | Live-browser capture / validation vectors | Anchors the model to real wallets; also the hardest artefact to produce | Never public |
| 6 | Tuned build & occupancy parameters | Turns ~50 Mkeys/s into ~80+ Mkeys/s per device | Never public |

**Why publish the economics and withhold the capability?** Because the
defensive community needs the cost model (to size recovery engagements, to
audit vendors' claims, to understand why mass exploitation is *not* happening
at scale), and the abuse community needs only the missing six components.
The first audience gets a complete chapter; the second gets a table of empty
slots they must fill independently — at a cost we estimate in the same
GPU-decades the economics describe. That asymmetry is the disclosure trade,
made deliberately and on the record.

## 9.7 What this means for holders and institutions

- **Mass exploitation is not the threat model.** At ~10¹⁷–10¹⁸ operations per
  funded target-week, Randstorm is an *engineered-recovery* problem, not a
  script-kiddie one. The panic headline ("millions of wallets vulnerable!")
  and the practical reality (individual, evidence-driven, expensive
  engagements) are both true.
- **Vendors claiming cheap mass recovery are lying** — either about the
  throughput (see §9.3) or about something else. The arithmetic in this
  chapter is a buyer's audit tool.
- **The only remediation remains moving funds** (§6). Every year a weak key
  stays funded is a year the hardware curve (§9.2) erodes its margin.
