# Quota Monitoring & Account Switching

> **How to use this module.** Load it with `resource-discipline.md` and `remote-compute.md`.
> It is **mandatory**: never let an account limit silently stall or degrade the job — detect
> it, and tell the user so they can switch accounts.

## Why this matters

Quotas and GPU availability are **per account**, not per session. Running many parallel
sessions draws down the *same* allowance. Colab's GPU access is dynamic and can be cut or
disconnected; Kaggle publishes a weekly GPU quota. When a limit is hit, the correct move is
to **tell the user**, who can switch to another account — not to wait, retry blindly, or
quietly fall back to something weaker.

## What to monitor

- **Colab** — GPU refusals ("you are not currently using a GPU" fallbacks), repeated
  disconnects, usage-limit warnings, sessions that start on CPU.
- **Kaggle** — remaining GPU hours (`kaggle quota`), push/run rejected for quota, queue
  placement.
- **Both** — session refused, idle-timeout kills, accelerator unavailable.

## Detection signals

- A session cannot get a GPU, or falls back to CPU.
- Repeated disconnects or "quota exceeded" errors.
- `kaggle quota` shows the weekly hours exhausted.
- A `kaggle kernels push` / job launch is rejected.

## Escalation protocol — tell the user

When **any** backend hits an account limit:

1. **Stop retrying on that backend.** Don't burn time or silently degrade quality.
2. **Report to the user, concisely:**
   - the backend and account;
   - the limit hit (e.g. *"Kaggle weekly GPU quota exhausted"*, *"Colab is not granting a
     GPU"*);
   - what was in flight and what failed;
   - **the ask** — *"switch to another account, or wait for the quota to reset"*.
3. **Keep going elsewhere** — continue on the other backend and on any sessions that still
   have quota. One limit must not stop the whole job.
4. **Record** the switch and which account/backend produced each artefact.

## Account rotation

- Keep every stage **resumable**, so work can move to a new account without loss.
- **Never hardcode credentials** — the user supplies and switches accounts.
- After a switch, **re-verify the GPU** and **re-stage inputs** (the remote filesystem is
  ephemeral).

## Guardrails

- **Never silently fall back to CPU or to the local machine.**
- **Never assume unlimited compute** just because many sessions are running.
- **Ask early** — a quick account switch beats a stalled run and a wasted quota.

## Sources

Google Colab and Kaggle usage/limit documentation; `remote-compute.md` and
`resource-discipline.md`.
