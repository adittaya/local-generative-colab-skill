# Remote Compute — Google Colab & Kaggle

> **How to use this module.** Load it alongside `architecture-and-cli.md` at the start of
> every run. It defines the two remote backends, how to choose between them, and the
> ephemeral-storage rule. Pick the backend per task.

## Two backends, one controller

The skill runs the same pipeline on either of **two free remote GPU backends**:

- **Google Colab** — driven by the `colab` CLI.
- **Kaggle** — driven by the `kaggle` CLI.

The local machine stays the controller and the source of truth in both cases. Choose the
backend per task (see "When to use which").

## Specifications & limits (free tiers)

| | **Google Colab** | **Kaggle** |
|---|---|---|
| Accelerator | 1× NVIDIA T4 | **2× NVIDIA T4** (32 GB total), or 1× P100 (16 GB); TPU v3-8 / v5e-8 / v6e-8 |
| GPU VRAM | ~16 GB | **~32 GB** (T4 ×2) |
| System RAM | ~12.7 GB | **~29 GB** |
| CPU cores | ~2 | 4 |
| Session length | variable; times out | **12 h** (CPU/GPU), 9 h (TPU) |
| Idle timeout | ~90 min | ~60 min |
| Weekly GPU quota | unpublished, dynamic | **30 h/week** (published), ~20 h TPU |
| Disk | ephemeral | 20 GB auto-saved `/kaggle/working` + non-persistent scratchpad |
| Control | `colab` CLI (interactive sessions) | `kaggle` CLI (push → run → pull outputs) |
| Persistence option | Google Drive mount | `/kaggle/working` (20 GB) + Kaggle datasets |
| Terms | general | **personal, non-commercial use** |
| Sign-up gate | Google account | Kaggle account + **phone verification** |

Kaggle's quota is **published and counted** (you can see remaining hours), which makes it
more predictable; Colab's is dynamic and can be cut without warning. Free accelerators on
both can be queued at busy times.

## When to use which (hybrid selection)

- **Use Kaggle when you need more power or longer runs.** 2× T4 (32 GB) and 29 GB RAM let
  you run bigger models and longer clips with more offloading headroom; the 12 h session and
  published 30 h/week quota suit long or batched jobs. Kaggle's `kernels push` also fits a
  **commit-and-run** workflow well.
- **Use Colab when you need interactive iteration**, a quick job, the interactive `colab`
  CLI, or a TPU — or when Kaggle's queue, phone-verification gate, or non-commercial terms
  are blockers.
- **Split across both** when one backend is quota-limited: e.g. iterate on Colab, run the
  heavy batch on Kaggle.
- **Run both in parallel.** Colab and Kaggle are independent, so split independent tasks
  across them and run them **concurrently in the background** to finish faster — while
  keeping one task per machine at a time (see `resource-discipline.md`).

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
