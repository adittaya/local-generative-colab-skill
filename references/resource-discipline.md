# Resource Discipline — Do Not Waste Remote Runtime

> **How to use this module.** Load it alongside `architecture-and-cli.md` and
> `remote-compute.md`, in every run. It governs *how* you spend the remote GPU time.

## Principle

Remote GPU time is scarce, quota-limited and shared. **Use it actively, and release it the
moment a task is done.** Quality is *not* time-limited — but idle runtime *is* waste. Work
deliberately and completely, then let the machine go.

## The local machine's job — and only its job

**All heavy work runs remotely.** The local machine is used **only** for:

- **saving files** — the home directory is the persistent source of truth;
- **running the controller scripts** — the CLI drivers that stage inputs, launch remote jobs
  and fetch results;
- **collecting and providing outputs** — pulling results back and handing them to the user.

Nothing heavy runs locally: no local inference, no local training, no local model loading,
no local reconstruction. That is the whole point of the remote backend.

## One task at a time, one machine at a time

- Work **sequentially**. Pick one task (e.g. image generation), run it, finish it, **release
  the machine**, then move to the next (e.g. video generation).
- Do **not** keep several heavy models resident at once, and do not interleave unrelated
  heavy jobs.
- After each task: **unload the model, free VRAM/RAM, clean temp**, then **stop the session**
  if no further work needs it.

## Stop cleanly when the work is done

- As soon as a task's outputs are pulled back to local, **release the GPU**: stop the
  runtime/session — or leave it running only if the *very next* queued task genuinely needs
  the same environment.
- Never hold a quota'd session open doing nothing. Idle timeouts and weekly quotas exist;
  idle time burns both.
- Clean scratch space, and record what was produced and where it went.

## No time limit — but no waste either

- There is **no working time limit**: do not rush, and do not degrade quality to save time.
  Take the steps needed for the best result.
- The discipline is the opposite of waste: **work actively while running, then release.**

## Practical checklist (run it around every task)

- [ ] Backend chosen; remote GPU verified.
- [ ] **Only** the models needed for *this* task loaded.
- [ ] Outputs pulled back to the local machine as they are produced.
- [ ] Checkpoints synced to local (or Drive / a Kaggle dataset).
- [ ] Model unloaded and VRAM/RAM freed after the task.
- [ ] Session stopped if nothing else needs it.
- [ ] Scratch cleaned; manifest updated.

## Why this matters

Both free tiers are quota-limited — Colab's is dynamic and can be cut, Kaggle's is 30 h/week
and counted — and free accelerators can queue. Wasting runtime burns quota and blocks the
next task, while holding idle sessions triggers timeouts and disconnects.

## Sources

Google Colab and Kaggle free-tier documentation and usage policies; `remote-compute.md`.
