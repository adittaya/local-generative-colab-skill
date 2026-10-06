# Example Project Summary — Worked Reference

> **How to use this module.** This is a **worked example** of the diagnosis → decision →
> system-finding summary the skill should produce and bank to the local machine at the end of
> a project. Read it as a template for the *format* and the level of specificity — not as
> instructions. End every project with a summary in this shape.

## What a good summary contains

1. **Project state** — what is banked locally, what is in flight, nothing lost.
2. **Diagnosis** — the exact error text, what it actually means, and the fix.
3. **Model system** — heavy → light, with a verdict per model and the locked stack.
4. **Quality-on-weak-hardware playbook** — how to get quality when the GPU is small.
5. **Hybrid ladder** — free → paid escalation, with costs.
6. **Operating runbook** — the repeatable steps for the next session.
7. **Pending** — whose court each open item is in.

## Example

### 1. Project state (banked locally, nothing lost)

- `CONCEPT.md` — direction locked, beat map, style pass, feature map, sync map, asset manifest.
- `ASSETS-PROMPT.md` — all generation prompts (continuity block + per-shot + audio).
- `REFERENCE-ANALYSIS.md` + `ref_analysis/` — reference probe: 29 cuts / ASL 4.7 s,
  −14.3 LUFS, word transcript, per-scene vision notes, palette, generated stills.
- `vo/` — 12/12 VO lines via Qwen3-TTS + durations.
- Remote scripts staged: video script, Kaggle kernel, probe, VO scripts, staging scripts.
- In flight: a CPU staging session downloading a video-model bundle (~26 GB).

### 2. Diagnosis — exact errors and what they mean

- **Colab GPU: `Service Unavailable` on the accelerator-assign call.** Meaning: free-tier
  usage-limit / capacity refusal (the CLI surfaces 412/503 opaquely; the server never says
  quota vs capacity). Confirmed usage-limit throttle. **Not fixable by flags, reinstalls,
  regions or runtimes** — it clears in ~12–24 h. CPU sessions are unaffected.
- **Kaggle: `Temporary failure in name resolution` → `outgoing traffic has been disabled`.**
  Meaning: the kernel runtime has no working DNS/internet despite `enable_internet: true`, so
  any in-kernel download fails. **Fix: offline-first weights** — Kaggle Models
  (`model_sources`) or weight datasets (`dataset_sources`), loaded from `/kaggle/input/...`.
- **Kaggle: `Permission 'kernels.get' was denied` + empty `datasets list --mine`.** Meaning:
  the CLI OAuth token is half-expired (says logged in, server rejects reads/writes). **Fix:
  legacy API key** → `~/.kaggle/kaggle.json` (non-expiring), or `kaggle auth login --force`.

### 3. Model system (heavy → light, with verdicts)

| Model | Weights | VRAM | Verdict |
|---|---|---|---|
| 22B-class video (LTX-2.x) | 30–46 GB | 32–80 GB | Out — needs A100/H100 |
| 14B video (Wan/Hunyuan/Mochi) | 10–40 GB | 20–24 GB+ | Paid only |
| 13B video bf16 / fp8 | 28 / 15.7 GB | 24 / 14–16 GB | Marginal (shard + offload) |
| 5B video | ~10 GB | ~20 GB | Paid only |
| **2B distilled fp8** | **4.5 GB** | **~5–8 GB** | **WORKHORSE** |
| 2B distilled bf16 | 6.3 GB | ~8–10 GB | Fallback |
| 1.3B specialists | ~2.5 GB | ~8 GB | Quality specialists (Apache 2.0) |
| SVD-XT / AnimateDiff | ~3–5 GB | ~8–16 GB | Offline fallbacks |

- Constant companion: the ~20 GB T5-XXL text encoder lives in **RAM, not VRAM**.
- **Quant note:** on a T4 (no fp8 cores), fp8 = storage savings, **not** speed. Recipe: fp8
  weights + diffusers layerwise casting + T5 on CPU + tiled VAE → ~8–10 GB total.

### 4. Quality-on-weak-hardware playbook (quality > time)

- **Fit:** lightest offload that fits; T5 on CPU; tiled VAE (512 / overlap 64, temporal
  16–32); Sage/flash attention; bf16.
- **Quality knobs:** steps in the model's band (distilled 8–10 / CFG 1.0; dev 20–50 / UniPC);
  correct guidance; native-res gen → Real-ESRGAN upscale; TeaCache ≤ 0.2; 3 seeds per hero
  shot and keep the best; prompt extension; **stills are the quality floor — iterate there**.
- **Polish:** frame interpolation to 24 fps, grain + cold→warm grade, sound design.
- **Discipline:** one resident model at a time; `del` + `empty_cache()` between stages; 2–5 s
  native clips then extend; save latents; skip `torch.compile` on eviction-prone runtimes.

### 5. Hybrid ladder (slow → high)

- **CPU-only:** validation/staging → SFX → stills → assembly/QA → micro-clips (proof only).
- **CPU+GPU ($0):** Colab T4 alone → Kaggle 2×T4 (2 parallel) → Colab+Kaggle (3 workers).
- **Paid:** Vast 3090 (~$0.14/hr) → RunPod 4090/A100 for the 22B tier.

### 6. Operating runbook (future sessions)

1. **Health probe first (30 s):** Colab T4 test + `kaggle datasets list --mine`. The working
   backend takes the heavy jobs.
2. **Offline-first weights:** download once (whichever backend has internet), mirror to
   Kaggle Models / a dataset, run offline after.
3. **Session hygiene:** one session, keep it busy, pull outputs immediately, **stop when
   done**. Idle = throttle trigger.
4. **Credentials:** legacy `kaggle.json` (no expiry); re-verify the Colab CLI each session.
5. **Tripwire:** both free backends down and a deadline burning → the smallest pre-decided
   Pay-As-You-Go pack.

### 7. Pending (whose court)

- **USER:** create a legacy Kaggle API key → `~/.kaggle/kaggle.json`.
- **AGENT:** Colab retry timer; staging bundle download → checksum → smoke test; fire
  generation the instant any GPU lands.
- **DECISION for user:** add a second free backend? A paid sprint if a deadline appears?

## Sources

Produced from a real project run (2026); backend failure modes cross-checked against the
Colab/Kaggle CLIs and `quota-and-accounts.md`.
