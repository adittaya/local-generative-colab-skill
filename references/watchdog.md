# Self-Healing Watchdog

> **How to use this module.** **Mandatory** whenever jobs run in the **background** — parallel
> sessions, Kaggle **versions**, or long Colab runs. A passive status poller is not enough: the
> watchdog must **act**.

## Why

Parallel background work is only safe if something is watching it. A status poller *reports*;
a **watchdog** *recovers, pulls, and alerts*. This is what keeps parallel sessions from
silently wasting quota or losing finished work.

## The four responsibilities

1. **Auto-pull on success** — the moment a job (or a `.done` unit) completes, fetch its
   outputs to **Tier 1 / Tier 2** (see `persistence-protocol.md`). Never wait for the session
   to end.
2. **Capture logs on failure** — on a failed run, save the **full logs** (stdout/stderr,
   `kaggle kernels logs`, session logs) to local **before the session is gone**.
3. **Re-push once on transient errors** — on a *transient* error (network blip, queue,
   eviction, DNS hiccup), **re-push/retry exactly once** automatically, then stop and report if
   it fails again.
4. **Alert on stalls / quota** — if a job **stalls** (no heartbeat past a threshold) or a
   **quota limit** is hit, stop and **tell the user** (see `quota-and-accounts.md`).

## The loop

```
for each background job:
  poll status + heartbeat
  ├─ success      → pull outputs (Tier 1/2), mark manifest "pulled"
  ├─ failed       → capture logs → classify
  │                  ├─ transient → re-push ONCE → back to poll
  │                  └─ fatal     → record + alert the user
  ├─ stalled      → alert the user (no progress > threshold)
  └─ quota hit    → alert the user, escalate an account switch
```

## Rules

- **Exactly one automatic re-push** per job — never an infinite retry loop.
- **Watch a heartbeat, not just status.** Track last-progress time; a job that is "running" but
  silent past the threshold is a **stall**.
- **Pull as you go** — any completed unit is pulled immediately, not at the end.
- **Logs are evidence** — always capture on failure, before the session dies.
- **Alert, don't spin** — on stall / quota / fatal, stop and tell the user with the specifics.
- **Idempotent re-push** — versioned Kaggle kernels/datasets make a re-push safe; re-pushing
  must never corrupt Tier 2.
- **Release the session** once its job is done — including the Kaggle **interactive** session
  (`remote-compute.md`).

## Alert format

```
job:      <name>            backend/version: <kaggle owner/kernel/version | colab session>
state:    success | failed | stalled | quota
evidence: <log path>  ·  last heartbeat <t>
action:   <pulled | re-pushed once | stopped>
needed:   <what the user must do — e.g. switch account, raise deadline>
```

## Sources

Built on `persistence-protocol.md` (pull-on-`.done`, logs, manifest) and
`quota-and-accounts.md` (limit detection and escalation).
