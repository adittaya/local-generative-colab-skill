# Remote Compute — Kaggle (primary) & Google Colab (fallback)

> **How to use this module.** Load it alongside `architecture-and-cli.md` at the start of
> every run. It defines the two remote backends, the **Kaggle-first** priority, and the
> ephemeral-storage rule.

## Two backends, one controller

The skill runs the same pipeline on either of **two free remote backends**:

- **Kaggle — the default, mandatory first choice.** CPU *and* GPU, 2× T4 (~32 GB), 29 GB RAM,
  12 h sessions, a published 30 h/week quota. Driven by the `kaggle` CLI.
- **Google Colab — the fallback.** 1× T4, ~12.7 GB RAM, dynamic quota. Driven by the `colab`
  CLI. Use it when Kaggle is queued, out of quota, or for a quick interactive test.

**Use Kaggle first for all light-to-heavy work; fall back to Colab.** The local machine stays
the controller and the source of truth in both cases.

## Specifications & limits (free tiers)

| | **Kaggle** (primary) | **Google Colab** (fallback) |
|---|---|---|
| Accelerator | **2× NVIDIA T4** (32 GB total), or 1× P100 (16 GB); TPU v3-8 / v5e-8 / v6e-8 | 1× NVIDIA T4 |
| GPU VRAM | **~32 GB** (T4 ×2) | ~16 GB |
| System RAM | **~29 GB** | ~12.7 GB |
| CPU cores | 4 | ~2 |
| Session length | **12 h** (CPU/GPU), 9 h (TPU) | variable; times out |
| Idle timeout | ~60 min | ~90 min |
| Weekly GPU quota | **30 h/week** (published), ~20 h TPU | unpublished, dynamic |
| Disk | 20 GB auto-saved `/kaggle/working` + non-persistent scratchpad | ephemeral |
| Control | `kaggle` CLI (push → run → pull outputs) | `colab` CLI (interactive sessions) |
| Persistence option | `/kaggle/working` (20 GB) + Kaggle datasets | Google Drive mount |
| Terms | **personal, non-commercial use** | general |
| Sign-up gate | Kaggle account + **phone verification** | Google account |

**Kaggle is the primary backend:** it offers both **CPU and GPU**, more VRAM and RAM, longer
sessions, and a **published, counted quota** (you can see remaining hours) — so it is faster,
more predictable, and suits light-to-heavy work. Colab's quota is dynamic and can be cut
without warning. Free accelerators on both can be queued at busy times.

## When to use which (Kaggle first)

- **Kaggle is the mandatory first choice.** Use it for **all light-to-heavy work** — it
  offers CPU *and* GPU, 2× T4 (32 GB) and 29 GB RAM for bigger models and longer clips, a 12 h
  session, and a published 30 h/week quota. `kaggle kernels push` fits a **commit-and-run**
  workflow well.
- **Colab is the fallback.** Use it only when Kaggle is queued, out of quota, blocked by its
  phone-verification gate or non-commercial terms, or for a quick interactive test — then move
  the real work back to Kaggle.
- **Run both in parallel when it helps.** They are independent, so split tasks across them
  and run **concurrently in the background** to finish faster — keeping one task per machine
  at a time (see `resource-discipline.md`).

## Kaggle CLI workflow

```bash
pip install kaggle
kaggle auth login                 # or set KAGGLE_API_TOKEN, or ~/.kaggle/access_token
kaggle --help

# run a notebook/script (kernel) on a chosen accelerator
kaggle kernels push -p <folder> --accelerator NvidiaTeslaT4
kaggle kernels push -p <folder> --no-run          # save a version without running
kaggle kernels status <owner/kernel>
kaggle kernels logs <owner/kernel>
kaggle kernels output <owner/kernel> -o <dir>     # pull outputs back to local
kaggle quota                                       # remaining GPU hours
```

- The run is driven by a `kernel-metadata.json` in the folder (generate a starter with
  `kaggle kernels init`).
- Accelerator IDs (as of late 2026): `NvidiaTeslaT4` (T4 ×2, default GPU), `NvidiaL4`,
  `NvidiaTeslaP100`, `TpuV5E8`, `TpuV6E8`.
- `kaggle kernels push` uploads **and runs**; fetch results with `kernels output`.

## Colab CLI workflow

Inspect the installed CLI first and adapt to its real syntax:

```bash
colab --version
colab --help
colab sessions
```

Use it to create/attach a remote session, execute scripts, and transfer files — see
`architecture-and-cli.md`.

## The ephemeral-storage rule (important)

**Treat BOTH remote filesystems as temporary scratch.** Sessions are recycled, files are not
guaranteed to survive, and neither backend is a reliable store.

- The **local machine's home directory is the source of truth** — persistent, precise, and
  the place the user actually keeps their work.
- Flow for every job:
  1. **Stage** inputs from the local machine to the remote backend.
  2. Do **all heavy work remotely** (never on the local machine).
  3. **Pull outputs back to the local machine** as soon as they exist — don't wait for the
     session to end.
  4. **Sync checkpoints** to local (or Google Drive / a Kaggle dataset) as you go, so a
     dropped session never costs work.
- **Never leave the only copy of a result on a remote session.**
- Keep a **manifest** of what was staged and what was returned, so the local tree stays the
  authoritative record.

For the full system — three tiers (local light truth, Kaggle heavy truth, sessions nothing),
`nohup` + `.done` markers + polling, determinism and `MANIFEST.json` — see
`persistence-protocol.md`.

## Guardrails

- **Verify the GPU** (`nvidia-smi`, CUDA, PyTorch) on whichever backend you use, before any
  heavy stage.
- **Kaggle terms are personal, non-commercial** — don't use it for client/commercial work.
- **Free accelerators may queue**, and both backends have an **idle timeout** — checkpoint
  often and design every stage to be **resumable**.
- **Don't assume persistence**: mount Drive (Colab) or write to `/kaggle/working` (Kaggle)
  only as a *convenience cache*, never as the system of record.
- **Pin the backend per run** and record which one produced each artefact.

## Sources

Kaggle notebook documentation and efficient-GPU-usage docs; Kaggle CLI docs (`kernels`
push/pull/output/status, accelerators) and repository; Google Colab documentation; and
published free-tier comparisons (2026).
