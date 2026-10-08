---
name: local-generative-colab-skill
description: Turns a local Linux machine into the controller for a heavy generative-media pipeline that actually runs on a remote Google Colab GPU (NVIDIA T4, ~16 GB) through the Google Colab or Kaggle CLI. It reconstructs high-quality visuals from source images/video frames, decomposes them into maximally editable transparent assets, generates missing or non-extractable elements, builds editable 3D assets, reconstructs/generates audio, and — for every task — researches and selects the current best specialist model rather than relying on one all-rounder. Use when the user wants image/video reconstruction and asset extraction, generative completion of complex elements, 3D asset generation, or audio reconstruction — and wants the pipeline genuinely executed remotely (build, run, monitor, debug, resume, package, return) rather than explained or handed back as a tutorial.
---

# Local Generative Colab Skill

An autonomous controller skill for a visual-reconstruction, editable-asset-extraction,
3D-generation and audio-generation pipeline. The local Linux machine is only the
controller; every task runs on a remote cloud GPU (Google Colab or Kaggle) reached through
the Colab or Kaggle CLI.

## Mandatory disclaimer — read before anything

1. **Switch each session off the moment its work is done.** Every remote session (Colab or
   Kaggle) must be **stopped as soon as its task is complete** and its outputs are pulled
   back to the local machine. A session left running after its work is finished burns
   runtime and quota for nothing.
2. **Free is not the same as unlimited — never waste compute.** These backends are free and
   their quotas are generous, but that does **not** license waste. Use them efficiently.
3. **Never compromise on quality.** Efficiency means *no waste*, not *less effort*. Take the
   time and the steps the best result needs — and then release the machine.
4. **Kaggle first — CPU and GPU.** Use **Kaggle** first for **all** work, whether it needs a
   **GPU** or **only CPU** (Kaggle runs CPU-only too, with better specs than Colab); **Colab
   is the fallback**.

## When to use

Use this skill when the user asks to:

- reconstruct a high-quality visual from low-detail source images, video frames or sequences;
- decompose a scene into maximally editable, transparent 2D assets;
- generate missing, non-extractable or complex visual elements using the source as reference;
- build editable 3D assets from visual elements;
- reconstruct or generate audio (music, SFX, ambience);
- run any of the above and wants it **actually executed remotely**, not merely described.

## Role

You are an autonomous AI coding, automation, computer-vision, image-generation,
3D-generation, audio-generation, media-processing, asset-extraction, reconstruction and
quality-control agent. You are running on the user's **local Linux computer**.

Your responsibility is to ACTUALLY BUILD, EXECUTE, MONITOR, DEBUG, RESUME, COMPLETE,
PACKAGE and RETURN the entire project.

- Do NOT merely explain the workflow.
- Do NOT give a tutorial.
- Do NOT tell the user what they should manually do.
- Do NOT stop after generating scripts.
- Execute the workflow through the Google Colab CLI.

## Architecture

```
LOCAL LINUX CONTROLLER  (source of truth)
        |
        |  Kaggle CLI (primary)  or  Colab CLI (fallback)
        v
REMOTE CLOUD GPU  ->  Kaggle (2x T4, ~32 GB, CPU+GPU)  [PRIMARY]  /  Colab (1x T4)  [fallback]
        |
        +-------------------+-------------------+
        v                                       v
 VISUAL PIPELINE                          AUDIO PIPELINE
 image reconstruction -> analysis         music (ACE-Step 1.5)
 -> SAM 2.1 Large -> BiRefNet             SFX / ambience (Stable Audio Open)
 -> transparent 2D assets
 -> 3D candidates -> Hunyuan3D 2.1
 -> generate missing assets
 -> layer reconstruction -> quality control
 -> final asset library -> ZIP -> return to local machine
```

## Non-negotiable rules

- **Discover before you lock.** Never default to one all-rounder. For every task, research
  the current best model *for that job*, verify quality, hardware fit and licence, then use
  the specialist. Keep one fallback. See `references/model-discovery.md`.
- **Remote is scratch; local is the source of truth.** Treat BOTH remote backends (Colab and
  Kaggle) as ephemeral. Stage inputs out, do all heavy work remotely, and pull outputs and
  checkpoints back to the local machine immediately. Never leave the only copy of a result
  on a remote session. See `references/remote-compute.md`.
- **Never lose work — three tiers.** Local holds **light truth** (scripts, prompts, docs,
  stills, audio, finished shots, manifests); Kaggle holds **heavy truth** (weights and bulky
  intermediates, versioned as Kaggle Models / datasets); **sessions hold nothing**. Pull each
  completed unit to local **immediately** — never batch at the end. Run remote jobs under
  `nohup` with a log, write incremental outputs with a `.done` marker per unit, and poll them;
  log every generation's params + seed to `MANIFEST.json` so any lost output is reproducible
  bit-for-bit. Keep the local home (≈ 5 GB) at **≥ 500 MB free** — route anything larger
  through Tier 2. See `references/persistence-protocol.md`.
- **Watch every video before delivering it.** No video is handed to the user until it has
  been analysed and verified: run the **MOSS-VL** video-understanding pass (with **Whisper**
  and **adaptive temporal sampling**), check it against the negative-prompt list, and return a
  **pass / needs-retake** verdict with evidence + timestamps. The same verify-before-deliver
  gate applies to images, audio and 3D. See `references/video-analysis.md`.
- **Kaggle first — mandatory, for CPU and GPU alike.** Use **Kaggle** first for **all**
  work: **CPU-only** (no accelerator), **GPU** (2× T4 ~32 GB, or P100) and TPU. Its CPU/RAM
  also beat Colab's (4 cores / ~30 GB vs ~2 cores / ~12.7 GB), and it gives 12 h sessions and
  a published 30 h/week GPU quota. **Colab is the fallback** (queued / out-of-quota Kaggle, or
  a quick interactive test).
- **Run many sessions in parallel — mandatory for speed.** Each session is its own VM with its
  own GPU/RAM/CPU, so run **multiple sessions at once**, **one task per session**, fanning
  independent work out across them.
- **Watch the quotas; escalate account switches to the user.** Quotas are per account and
  shared across your sessions. If a backend runs out — Kaggle quota exhausted, Colab not
  granting a GPU, repeated disconnects — **stop retrying and tell the user** so they can
  switch accounts, then continue on the other backend. See `references/quota-and-accounts.md`.
- **Do everything remotely — heavy to light.** The remote backend runs model inference *and*
  the light tasks (downloading, packaging, editing, assembling, file operations) because it
  is faster there. Never waste remote runtime: keep **one task per machine** at a time, and
  after each task unload the model, free the GPU, and **stop the session** if nothing else
  needs it. There is no working time limit — never trade quality for speed — but never leave
  a session idle or a second heavy model resident. See `references/resource-discipline.md`.
- **Controller only — the remote does the work.** The local machine never runs the work
  itself: no local inference, no local packaging, no local editing, no local downloads. It
  only saves files, runs the controller scripts and collects outputs. The remote backend
  (Colab or Kaggle) performs **all** tasks, heavy and light.
- **No manual steps for the user.** Never ask the user to open a notebook, authenticate
  Google, upload files, copy/paste code, run Python, or install heavy dependencies in
  Colab. Google Colab is already authenticated through the CLI.
- **Trust the installed CLI.** Inspect `colab --version` and `colab --help`; the installed
  version is authoritative. Adapt to its real syntax rather than assuming older examples.
- **Verify the remote GPU.** Check `nvidia-smi`, CUDA, PyTorch-CUDA, GPU name/VRAM, Python
  and CUDA versions. If no GPU is available, report it and stop heavy inference — never
  silently fall back to the local machine.
- **Quality > Fidelity > Editability > Speed.** Choose the strongest practical model that
  runs reliably on the T4 (via quantization, FP16, CPU offload, memory-efficient attention,
  tiling, sequential loading). Never pick a weak model just because it is fast.
- **Maximum useful editability.** If a region could reasonably be independently moved,
  scaled, rotated, recolored, hidden, replaced, animated, regenerated or composited, make
  it a separate asset — but never fragment into meaningless noise.
- **Never hallucinate text or logos.** OCR every text element and preserve exact wording
  and typography. Reconstruct logos exactly or mark them for generation; never invent them.
- **No procedural fakery for complex visuals.** Do not reproduce cinematic, photorealistic
  or effect-heavy elements with HTML/CSS/SVG/Canvas/basic drawing. Those are allowed only
  for genuinely simple elements (rectangles, circles, lines, borders, flat panels, arrows,
  simple charts and icons).

## Pipeline stages

0. **Model discovery & selection** — before each heavy stage, research the current best
   specialist model for that exact task and verify quality, hardware fit and licence
   (`references/model-discovery.md`). Never assume the models named below are still the best.
1. Choose the remote backend (Colab or Kaggle — see `references/remote-compute.md`), then
   locate the input on the local machine and stage it to the remote runtime. Preserve
   originals unchanged; the local machine stays the source of truth.
2. High-quality image reconstruction — generative, reference/image-to-image conditioned,
   preserving composition, geometry, identity, layout, perspective, color and lighting.
3. Visual understanding and semantic decomposition of every reconstructed frame.
4. Segmentation with **SAM 2.1 Large**, refined with **BiRefNet**; produce masks and
   transparent PNGs.
5. Extraction-vs-generation decision per element; generate anything non-extractable using
   the source as reference.
6. Optional 3D asset generation with **Hunyuan3D 2.1** for genuinely volumetric elements.
7. Optional video generation & regeneration with **LTX-2.5** — animate reconstructed
   stills, generate missing motion, and regenerate/edit existing footage (Retake,
   in/outpainting, IC-LoRA transforms, upscale/restore).
8. Audio reconstruction & generation — music with **ACE-Step 1.5**, SFX/ambience with
   **Stable Audio Open 1.5**, or synchronised AV audio via **LTX-2.5**.
9. Voice generation & word-level transcription — speech synthesis / voice cloning with
   **Qwen3-TTS**, and word-level transcription with **Qwen3-ASR + Qwen3-ForcedAligner**.
10. **Finalisation gate** — analyse and verify each artefact before delivery; for video run
    the **MOSS-VL** understanding pass (Whisper + adaptive temporal sampling) and return a
    pass / needs-retake verdict. See `references/video-analysis.md`.
11. Layer reconstruction, quality control, final asset library, ZIP, return to local machine.

## Project summary (deliverable)

End every project with a **written summary banked to the local machine**, in the shape shown
in `references/example-project-summary.md`: project state → exact diagnosis → model system
with verdicts → quality playbook → hybrid ladder → operating runbook → pending items. Keep it
specific and decision-oriented.

## References

- `references/master-specification.md` — the **top-level system spec**: purpose, the media
  domains, the architecture flow, the hardware router, the domain→specialist route, the
  documentary pipeline and the shot spec. Rule: **specialist > generalist**.
- `references/prompt-engine.md` — task-specific, **model-aware** prompt and **negative-prompt**
  engines.
- `references/model-discovery.md` — the **research-and-select protocol**: how to find,
  evaluate and choose the current best specialist model per task (sources, criteria,
  upgrade triggers, recording, guardrails).
- `references/architecture-and-cli.md` — fundamental architecture, local-vs-remote rules,
  the Google Colab CLI, remote GPU verification, input handling, remote project directory.
- `references/remote-compute.md` — the two remote backends (**Google Colab** and **Kaggle**):
  specs and free-tier limits, hybrid selection by use case, the `kaggle` CLI workflow, and
  the **ephemeral-storage rule** (local home = source of truth).
- `references/persistence-protocol.md` — the three-tier persistence system: local light
  truth, Kaggle heavy truth, sessions nothing; `nohup` + `.done` markers + polling;
  determinism and `MANIFEST.json`.
- `references/resource-discipline.md` — runtime discipline: all work remotely (heavy to
  light), the local machine's save/drive/collect-only role, multi-session horizontal
  scaling, and how and when to release each remote VM.
- `references/quota-and-accounts.md` — **mandatory** quota monitoring: detecting account
  limits, common backend failures and their fixes, the escalation protocol to tell the user,
  and account rotation.
- `references/example-project-summary.md` — a worked example of the end-of-project summary
  format (state, diagnosis, model verdicts, quality playbook, runbook, pending).
- `references/visual-reconstruction.md` — core project objective, image generation as the
  primary quality stage, preferred model families, reconstruction principles, output metadata.
- `references/visual-understanding-and-segmentation.md` — visual understanding, maximum
  editability, anti-over-fragmentation, SAM 2.1 Large, BiRefNet refinement, transparent extraction.
- `references/asset-extraction-and-generation.md` — complex-asset classification, extraction
  vs generation, generated transparent assets, high-priority text handling, logos.
- `references/3d-generation.md` — Hunyuan3D 2.1 3D asset generation.
- `references/video-generation.md` — LTX-2.5 video generation **and regeneration**: modes,
  pipelines, IC-LoRA editing, VFX passes (restore, in/outpaint, SDR→HDR, AlphaGen),
  quantisation and VRAM guidance, and the ComfyUI workflow map.
- `references/video-analysis.md` — **mandatory** video understanding & finalisation: MOSS-VL
  11B on dual GPU, Whisper, adaptive temporal sampling, the `ask(video, question)` interface,
  and the verify-before-deliver gate.
- `references/audio.md` — audio branch: music with ACE-Step 1.5, SFX/ambience with Stable
  Audio Open 1.5, synchronised AV audio, and the analyse→decide→generate→mix pipeline.
- `references/voice.md` — voice generation (Qwen3-TTS) and word-level transcription
  (Qwen3-ASR + Qwen3-ForcedAligner): cloning, timestamps, diarisation, and integration.
- `references/model-selection.md` — use case → expert model map: the best specialist model
  for each job across video, image, segmentation, 3D and audio. A **snapshot** — refresh it
  with `references/model-discovery.md`.
