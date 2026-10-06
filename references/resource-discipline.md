# Resource Discipline — Work Remotely, and Do Not Waste the Runtime

> **How to use this module.** Load it alongside `architecture-and-cli.md` and
> `remote-compute.md`, in every run. It governs *where* work runs and *how* you spend the
> remote runtime.
>
> ⚠ **Mandatory — the single most important rule here: switch a session OFF the moment its
> work is done.** Free is not unlimited; never waste compute, and never compromise on
> quality.

## Principle

The remote machine is faster at **everything** — not only heavy inference. So do **all**
work remotely, heavy *and* light. And because remote runtime is scarce and quota-limited,
use it actively and **release it the moment a task is done**. Quality is not time-limited —
idle runtime is waste.

To finish sooner, **use both backends in parallel**: Colab and Kaggle are independent, so
split the work across them and run jobs concurrently in the background (see below).

## What runs remotely — everything

Run on the remote backend, from heavy to light:

- **model inference** — image, 3D, video, audio, voice, segmentation;
- **downloading** — models, weights, checkpoints, datasets, reference assets;
- **packaging** — zipping, archiving, bundling the final asset library;
- **editing and assembling** — compositing, muxing, format conversion, ffmpeg passes,
  subtitle burn-in, audio mixing;
- **file operations** — moving, renaming, hashing/checksums, manifest building, uploads.

If a task can run remotely, it **runs remotely**. It is faster there.

## The local machine's job — and only its job

- **saving files** — the home directory is the persistent source of truth;
- **running the controller scripts** — the CLI drivers that stage inputs, launch remote jobs
  and fetch results;
- **collecting and providing outputs** — pulling results back and handing them to the user.

Nothing else. **No local inference, no local packaging, no local editing, no local
downloads.** The local machine is the controller and the store — not a worker.

## Parallel sessions — scale horizontally (mandatory for speed)

Each session — Colab or Kaggle — gets its **own VM with its own GPU, VRAM, RAM and CPU**.
Sessions do **not** share memory. So to finish faster, run **multiple sessions in parallel**:
several Colab sessions *and* several Kaggle sessions at once, each taking a different task or
a shard of the same task.

- **One task per session.** Each session is an independent worker — give it exactly one task.
- **Fan out** independent work — per image, per frame, per asset, per model — across sessions.
- **Files are local to a session** and vanish with it (`/content` on Colab; the scratchpad on
  Kaggle). Pull results back to local (or Drive / `/kaggle/working`) as they are produced.
- **Not dedicated hardware.** The provider may virtualise and share physical GPUs, so
  per-session throughput varies.
- **Quotas are per account, shared across your sessions.** More sessions do **not** mean
  unlimited compute — they draw down the same allowance. Scale only as far as the account
  allows, and watch the limits — see `quota-and-accounts.md`.
- After each session's task: **unload the model, free that VM, and stop the session** if
  nothing else needs it.

## Mandatory: switch each session off when its work is done

- **As soon as a task's outputs are pulled back to local, stop that session.** Do not leave
  it running "just in case" — an idle session burns runtime and quota for nothing.
- Keep a session alive **only** if the *very next* queued task genuinely needs the same
  environment — and stop it the moment that task finishes.
- Never hold a quota'd session open doing nothing. Idle timeouts and weekly quotas exist;
  idle time burns both.
- Clean scratch space, and record what was produced and where it went.
- **Free ≠ unlimited.** The backends are free with generous quotas, but that never licenses
  waste. Be efficient — **without compromising quality**.

## No time limit — but no waste either

- There is **no working time limit**: do not rush, and do not degrade quality to save time.
  Take the steps needed for the best result.
- The discipline is the opposite of waste: **work actively while running, then release.**

## Practical checklist (run it around every task)

- [ ] Backend chosen; remote GPU verified.
- [ ] **Only** the models needed for *this* task loaded.
- [ ] All work — including downloads, packaging and assembling — done remotely.
- [ ] Independent work **fanned out across multiple parallel sessions** (several Colab and
  several Kaggle sessions at once), one task per session.
- [ ] Quotas/availability checked per account; **user told** if any backend is over limit
  (see `quota-and-accounts.md`).
- [ ] Outputs pulled back to the local machine as they are produced.
- [ ] Checkpoints synced to local (or Drive / a Kaggle dataset).
- [ ] Model unloaded and VRAM/RAM freed after the task.
- [ ] Session stopped if nothing else needs it.
- [ ] Scratch cleaned; manifest updated.

## Why this matters

Both free tiers are quota-limited — Colab's is dynamic and can be cut, Kaggle's is 30 h/week
and counted — and free accelerators can queue. Wasting runtime burns quota and blocks the
next task, while holding idle sessions triggers timeouts and disconnects. Doing even the
light tasks remotely also keeps the local machine free and the pipeline fast.

## Sources

Google Colab and Kaggle free-tier documentation and usage policies; `remote-compute.md`.
