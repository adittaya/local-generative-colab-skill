# Persistence Protocol — How We Never Lose Work

> **How to use this module.** Load it with `remote-compute.md` and `resource-discipline.md`,
> in every run. It defines **where every artefact lives** and how work survives session death.

## The rule

**Local holds light truth, Kaggle holds heavy truth, sessions hold nothing.**

## TIER 1 — Local source of truth (the working folder, ~MBs)

Scripts, prompts, CONCEPT/ASSETS docs, stills, VO wavs, finished shots, manifests.

- **Pulled from remote immediately, per completed unit — never batched at the end.**
- **Disk budget: the local home directory is ≈ 5 GB. Always keep at least 500 MB free** —
  never let staging fill the disk. Anything that would breach that (large weights, bulky
  intermediates, big renders) goes to **Tier 2**, not to local.
- This is the system of record: if every remote tier dies, Tier 1 reconstructs the work.

## TIER 2 — Kaggle persistent storage (weights + bulky intermediates)

- **Kaggle Models** (`kaggle models create`) for weight bundles.
- **Kaggle datasets** (`kaggle datasets version`) for bulky intermediates.
- Versioned, free, and survives all sessions. Push large bundles **directly from Colab**
  when the local disk cannot hold them (e.g. a ~26 GB model bundle).

## TIER 3 — Remote sessions (pure scratch)

- **Assume death at any moment** — eviction, throttle and DNS death all happen.
- Every remote job **MUST**:
  1. run under **`nohup` with a log file**;
  2. write outputs **incrementally**, with a **`.done` marker per unit**;
  3. be **polled** — the poller pulls each `.done` unit to Tier 1 / Tier 2 **immediately**.

## Determinism

Every generation logs **full parameters + seed** into `MANIFEST.json`. Any lost output is
therefore **reproducible bit-for-bit** from the Tier 1 records.

## Manifest (live inventory)

`MANIFEST.json`, updated on **every pull**. Statuses advance:

`planned → staged → complete(remote) → pulled → verified`

## Why this matters

Free backends evict and throttle, the local disk is small, and sessions are ephemeral. Three
tiers mean **no single failure loses work**, and any lost remote output can be regenerated
from the Tier 1 record.

## Checklist

- [ ] Local home has **≥ 500 MB free** before staging (home is ≈ 5 GB).
- [ ] Every remote job runs under `nohup` with a log file.
- [ ] Incremental outputs with a `.done` marker per unit.
- [ ] The poller pulls each `.done` unit to Tier 1 / Tier 2 at once.
- [ ] Heavy weights / bulky intermediates versioned to Kaggle Models or datasets.
- [ ] Every generation's params + seed written to `MANIFEST.json`.
- [ ] Manifest statuses advanced on every pull.

## Sources

Project-derived persistence protocol (2026), built on the ephemeral-storage rule in
`remote-compute.md`.
