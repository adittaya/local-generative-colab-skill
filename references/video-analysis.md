# Video Analysis & Finalisation — Watch Before You Deliver

> **How to use this module.** **Mandatory.** Load it (a) whenever a video must be
> *understood* — analysis, QA, search, event extraction — and (b) as the **finalisation gate
> before any video is handed to the user**. No generated video leaves the pipeline unwatched.

## 1. Why

A video must be **watched**, not merely generated. This module defines how to understand video
properly and how to verify it before delivery. The same *verify-before-deliver* discipline
applies to every modality (image, audio, 3D); video is the flagship case.

## 2. Model

**MOSS-VL 11B** (OpenMOSS) — an open-weight vision-language model for **long-form video
understanding**. 11B parameters, BF16, **256K context**, cross-attention architecture
(XRoPE), native dynamic resolution and interleaved image/video inputs.

| Variant | Use |
|---|---|
| **MOSS-VL-Instruct** | offline long-video understanding, temporal reasoning, event localisation — the default for analysis |
| **MOSS-VL-Realtime** | real-time streaming video interaction (timestamped frames) |
| **MOSS-VL-Base** | continued pre-training / fine-tuning |

A **quantised NF4 build (~24 GB)** exists, plus an SGLang-Omni serving package. Defaults:
video FPS 1.0, max 256 frames, vision patch 16, temporal patch 1.

## 3. Dual-GPU layout (Kaggle 2×T4, 32 GB)

With Kaggle's two T4s, **shard the 11B model across both GPUs for inference**
(`device_map="auto"`) rather than squeezing everything onto one.

```
Kaggle
├── GPU 0 ──┐
│           ├── MOSS-VL 11B   (sharded across both)
├── GPU 1 ──┘
├── video decoder
├── audio → Whisper
├── timestamp / event index
└── temporal retrieval
```

## 4. Quality-first: optimise for *evidence density*, not FPS

| Component | Quality-first setting |
|---|---|
| Video sampling | **adaptive**, not fixed |
| Scene changes | detect explicitly |
| Important events | dense sampling |
| Audio | **full transcription** |
| Context | keep timestamps |
| Long videos | hierarchical processing |
| Ambiguous events | **re-inspect the interval** |
| Final answer | **evidence + timestamps** |
| GPU | **both GPUs** |
| Quantization | BF16 if it fits; otherwise FP8 / NF4 |

## 5. Adaptive temporal sampling — the key trick

Never turn a 60-minute video into 3,600 evenly spaced frames. Sample coarse-to-fine:

```
60 min
  ↓  coarse scan
~100–300 candidate events
  ↓  identify important / ambiguous sections
dense inspection of those sections
  ↓  cross-reference surrounding context
final reasoning
```

This gives the model far more *useful* visual information without blowing the context window.
(Published long-video work — EcoFrame, AdaFocus, DIG, FOCUS, LENS, AVP — consistently shows
coarse-to-fine, uncertainty-triggered re-inspection beats uniform sampling.)

## 6. Hybrid system (best quality)

**MOSS-VL 11B + Whisper + temporal retrieval + second-pass verification.**

- **Whisper** fully transcribes the audio (with timestamps).
- **Temporal retrieval** indexes events and timestamps so intervals can be re-found.
- **Second pass:** hint the model — *"at 17:32–17:48 something relevant happened"* — and have
  it **re-examine that interval at high resolution** rather than trusting its first read.

That is much closer to actually watching the video.

## 7. `ask(video, question)` interface

Expose one entry point: **`ask(video, question)`** → an answer **with evidence and
timestamps**. Internally: decode → transcribe (Whisper) → adaptive-sample → reason (MOSS-VL)
→ verify (second pass) → answer.

## 8. Mandatory finalisation gate (before delivery)

**Before any video is delivered to the user**, run this pass and report:

1. **What it shows** — a short factual description.
2. **Defects** — flicker, temporal instability, identity drift, face deformation, extra limbs,
   warped hands, duplicate subjects, unstable background, geometry deformation, texture
   swimming, camera jitter, lighting flicker, A/V desync, text artefacts (cross-check the
   negative-prompt list in `prompt-engine.md`).
3. **Evidence + timestamps** for each check (so the verdict is auditable).
4. **Verdict** — **pass**, or **needs retake** with the exact time range; retake the flagged
   segment with LTX-2.5 **Retake** (`video-generation.md`) rather than regenerating the whole
   clip.

## 9. Applies to all outputs

Run the same verify-before-deliver gate on **images, audio and 3D**: inspect the artefact,
check it against its negative-prompt list, and record a pass / needs-retake verdict with
evidence. Video is simply the case that most needs it.

## Sources

MOSS-VL (OpenMOSS) repository, project page and Hugging Face model cards; long-video
understanding research (EcoFrame, AdaFocus, DIG, FOCUS, LENS, AVP, LVNet); Whisper; and the
prompt/negative-prompt engines in `prompt-engine.md`.
