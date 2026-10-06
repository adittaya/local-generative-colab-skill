# Resource Discipline — Work Remotely, and Do Not Waste the Runtime

> **How to use this module.** Load it alongside `architecture-and-cli.md` and
> `remote-compute.md`, in every run. It governs *where* work runs and *how* you spend the
> remote runtime.

## Principle

The remote machine is faster at **everything** — not only heavy inference. So do **all**
work remotely, heavy *and* light. And because remote runtime is scarce and quota-limited,
use it actively and **release it the moment a task is done**. Quality is not time-limited —
idle runtime is waste.

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
- [ ] All work — including downloads, packaging and assembling — done remotely.
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
