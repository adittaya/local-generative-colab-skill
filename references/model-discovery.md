# Model Discovery & Selection Protocol

> **How to use this module.** Load it whenever a stage is about to lock in a model, and
> re-run it periodically. It tells you to research the current best model *for the specific
> task* rather than defaulting to one all-rounder.

## Purpose

This skill must never be stuck on a fixed, ageing model set. For every task, **research the
current best model for that exact job**, verify it fits the hardware and the licence, and use
the specialist instead of a single all-rounder. Re-run this protocol whenever a stage is about
to lock in a model, and periodically to upgrade.

## Core principle

**"Best for this task" beats "one model for everything."** An all-rounder is a fallback, not
the default. If a specialist model does a specific job better — higher quality, lower VRAM,
cleaner licence — use it for that job.

## The discovery loop

1. **Frame the task** precisely: modality, input, output, constraint (e.g. text-to-video,
   word-level ASR, voice cloning, image-to-3D, matting, music, segmentation).
2. **Search current sources** (below) for the leading models for that task.
3. **Collect candidates** — name, version, release date, size, licence, hardware path.
4. **Evaluate** against the criteria below.
5. **Decide** — pick the top model that fits hardware **and** licence; keep one fallback.
6. **Record** the choice (see the model-card note).
7. **Run**, then **revisit** if quality or constraints disappoint.

## Where to research

- **Hugging Face** — model pages, trending models, and the **Open ASR Leaderboard**.
- **Leaderboards & arenas** — TTS Arena (voice), Artificial Analysis (video/LLM), VBench
  (video), Papers-with-Code / arXiv (SOTA claims).
- **Official repositories** — GitHub release notes and changelogs (authoritative dates and
  capabilities).
- **Ecosystem support** — ComfyUI / Diffusers / vLLM integration (maturity and tooling).
- **Community** — benchmark write-ups, Reddit/Discord (real-world VRAM and quality).
- **The licence text on the model card** — always the authoritative source.

## Evaluation criteria (in priority order)

1. **Quality for the exact task** — use the task-specific metric, not a generic MOS.
2. **Hardware fit** — VRAM **and** host RAM; is there a quantised/offloaded path?
3. **Licence** — commercial use, region exclusions, fine-tuning allowed.
4. **Recency** — release/update date; has it been superseded?
5. **Ecosystem** — official code, ComfyUI/Diffusers nodes, LoRAs, community workflows.
6. **Fine-tunability** — LoRA / IC-LoRA / training code available.
7. **Speed / throughput** — wall-clock per unit of output.
8. **Reproducibility** — weights and configs public and pinned.

## When to upgrade or replace a model

- A newer model shows clearly better task quality.
- A better licence appears (e.g. Apache 2.0 replacing a restricted one).
- A lower-VRAM quant makes a stronger model fit the hardware.
- The current model is superseded, deprecated, or its repo is gated/unavailable.
- A task-specific specialist emerges that beats the generalist.

## Recording a choice (model-card note)

For every selected model, write and store alongside the stage metadata:

```
name · version · release date · params/size · licence · source URL ·
why chosen · hardware path (quant/offload) · fallback model
```

## Guardrails

- **Verify the licence yourself** before any commercial use — never rely on a summary.
- **Don't trust vendor benchmarks alone** — prefer independent or leaderboard results.
- **Keep a fallback** — never let one unavailable model stop the pipeline.
- **Re-check before production runs**, not only at project start.
- **Respect region exclusions** — some licences exclude specific countries.
- **Pin the versions you validated**; don't let them silently drift.

## Ready-to-use research prompt

> You are selecting a model for **<TASK>**. Research the current best open/local models for
> it as of **<DATE>**. For each candidate return: name, version, release date, parameters,
> licence (commercial? region limits? fine-tuning allowed?), VRAM/hardware path (quantised
> or full precision), task-specific quality evidence **with sources**, and ecosystem support.
> Rank the candidates and recommend **one primary + one fallback**, with the trade-offs and
> the main risk in each.

## Relationship to `model-selection.md`

`model-selection.md` is the **cached result** of this protocol — a fast starting map. Treat it
as a snapshot: refresh it with this protocol before locking a model, and update it when the
field moves. This file is the *process*; `model-selection.md` is the *current answer*.

## Sources

Hugging Face (model cards, Open ASR Leaderboard), TTS Arena, Artificial Analysis, VBench,
Papers-with-Code / arXiv, official model repositories, and community benchmark write-ups.
