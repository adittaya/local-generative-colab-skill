# Video Generation & Regeneration

This module adds the **time dimension** to the pipeline: turning reconstructed stills into
motion, and generating or regenerating existing footage. The primary model is
**LTX-2.5** (Lightricks) — the latest LTX open-weights audio-video "world" model.

## 1. Model

LTX-2.5 is a **22B asymmetric dual-stream diffusion transformer**: a 4096-wide video path
and a 2048-wide audio path joined by cross-modal attention. It generates **synchronised
video and audio in a single pass**, up to **4K at 50 fps**, with clips up to ~20 s on the
Fast tier. It is the newest LTX release (2026); Lightricks' LTX-2 toolkit added LTX-2.5
support in `v1.2.0` (11 Aug 2026), and community coverage ran on into October 2026.

What is new versus LTX-2.3:

- **Diffusion Fidelity Rendering (DFR)** — allocates render compute by scene complexity
  instead of evenly across frames; holds detail in crowds, fast motion and dense shots.
- **New diffusion video decoder (DiffVAE)** — replaces plain VAE decoding; sharper faces,
  legible on-screen text, fewer motion smears.
- **Native multishot** — one generation yields multiple connected shots that hold
  character, environment, lighting, voice and style across cuts.
- **Custom Gemma 4 12B text encoder** + **prompt enhancer** — tracks multiple subjects,
  actions, lighting and camera direction through complex prompts.
- **Auto duration** — predicts clip length from the described action.
- **Substantially improved distilled model** — near-full-model quality in a smaller,
  faster checkpoint.
- **Native 4K HDR and RAW/EXR** workflow for professional finishing.
- **Raw pre-trained checkpoint** (no SFT) for domain adaptation.

**Licence:** LTX-2.x Community License — free for organisations under ~$10M annual
recurring revenue (larger firms negotiate), no mandatory output branding, on-prem and edge
deployment allowed, fine-tuning allowed. The main `Lightricks/LTX-2.5` Hugging Face repo is
gated (sign in and accept the licence).

## 2. Capability map

### 2.1 Generation

- **Text-to-Video (T2V)** — video + synchronised audio from a prompt.
- **Image-to-Video (I2V)** — animate a still; prompt describes what happens *next*.
- **First-Frame / Last-Frame-to-Video (FLF2V)** — motion between two key images.
- **Audio-to-Video (A2V)** — video timed to a supplied audio track (held fixed).
- **Text-to-Audio (T2A)** — audio only, via the audio branch.
- **Native multishot** — a full sequence of connected shots in one generation.
- **Auto duration** — model picks the clip length; omit `--num-frames`.

### 2.2 Regeneration & editing

This is the "regeneration" branch — modify existing footage without rebuilding it.

- **Retake** — regenerate only a time window `[start_time, end_time]` of an existing
  video from a new prompt. Everything outside the window is left bit-for-bit identical.
  Modes: `replace_video`, `replace_audio`, `replace_audio_and_video`.
- **Extend** — continue a video from its last frame with a new prompt; no seams or cuts.
- **Keyframe interpolation** — generate a smooth transition between two keyframe images.
- **Inpainting** — replace a masked region (remove/replace objects) using the
  In-Outpainting IC-LoRA; mask-aware, blends via a Laplacian pyramid.
- **Outpainting** — extend the canvas beyond the original frame (sides, top, bottom).
- **V2V IC-LoRA transforms** — change identity, appearance or style while following the
  source footage (deblur, colorization, day-to-night, decompression, and more).
- **Dub-It** — rephrase dialogue while matching speaker identity and lip movement.
- **Union Control** — condition on depth, Canny edges or pose skeletons (one checkpoint
  accepts all three).
- **Motion Track** — animate a still image along drawn motion paths.
- **Ingredients** — generate a scene from a reference sheet of characters, props and
  locations.
- **Upscale / Refine / Restore** — Tiled Fusion upscaling to HD/4K/8K; Refine (sharpen and
  rebuild detail in blurry/compressed footage); Restore (revive archival footage).
- **HDR family** — SDR→HDR, HDR inpainting, HDR image-to-video, tiled SDR→HDR and
  HDR→HDR; native 16-bit EXR in and out.
- **Layout to Render** — turn a 3D clay render / blocking animation into a finished shot,
  following its camera movement, composition and object placement.
- **AlphaGen (beta)** — generate frame-aligned grayscale **alpha mattes** for compositing,
  including hair and smoke.
- **360° outpainting**.

### 2.3 Fine-tuning

- **LTX Trainer** reproduces LoRAs and IC-LoRAs; the large majority of LTX-2.3 LoRAs run on
  LTX-2.5 unchanged (validate before production).
- The **raw pre-trained checkpoint** adapts toward new domains (robotics, synthetic AV,
  digital twins, private domain models).

## 3. Pipelines (from the LTX-2 repository)

| Pipeline | Purpose |
|---|---|
| `DistilledPipeline` | Fastest text/image-to-video — the starting point |
| `DFRPipeline` | Production-quality T2V/I2V; keyframes + detailing pass + optional temporal upscaling |
| `TI2VidTwoStagesPipeline` / `...HQPipeline` | Guided two-stage T2V/I2V with CFG/STG and 2× upsampling |
| `ICLoraPipeline` | Video-to-video and image-to-video transformations |
| `KeyframeInterpolationPipeline` | Interpolate between keyframe images |
| `A2VidPipelineTwoStage` | Audio-to-video conditioned on an input audio file |
| `RetakePipeline` | Regenerate a specific time region of an existing video |
| `HDRICLoraPipeline` | V2V with HDR IC-LoRA output (EXR-ready) |
| `DubItPipeline` | Rephrase while matching speaker identity and lip movement |

Key model files: a **transformer** (distilled or dev), the **Gemma 4 12B** text encoder
(projection bundled), the **video VAE** (DiffVAE or lighter Conv VAE), the **audio VAE**,
the **spatial upscaler**, and — for DFR — the **detailing IC-LoRA**.

## 4. Hardware & quantisation

Official Lightricks checkpoints do **not** fit 16 GB: the smallest official pair
(NVFP4 transformer + INT8 Gemma 4 encoder) is already ~34 GB combined. Reality by tier:

| VRAM | Path | Notes |
|---|---|---|
| **16 GB** | Community **GGUF only** | No official combination fits — use a GGUF transformer + a GGUF Gemma 4 encoder with the ComfyUI-GGUF node |
| 24 GB | Official INT8/NVFP4 (tight), or GGUF Q6_K/Q8_0 | INT8 transformer + INT8 encoder is workable |
| 32 GB | Official INT8/NVFP4 comfortably; BF16 with offloading | Lightricks' stated minimum |
| 48 GB+ | Official BF16 full precision | Everything resident, no offloading |

GGUF transformer ladder (distilled, community uploads such as `realrebelai/LTX-2.5_GGUFs`,
`Abiray/LTX-2.5-Distilled-GGUF`, `elix3r/LTX-2.5-22b-distilled-GGUF`):

| Quant | Size | Fits 16 GB? | Quality |
|---|---|---|---|
| Q8_0 | ~22–24 GB | no | near-lossless |
| Q6_K | ~18–19 GB | no | excellent |
| Q5_K_M | ~16–18 GB | marginal | very good |
| Q4_K_M | ~14–16 GB | yes | recommended balance |
| Q3_K_M | ~11–13 GB | yes | usable, softer detail |
| Q2_K | ~8–9 GB | yes | heavy loss — last resort |

The **text encoder** is the other half of the budget: Gemma 4 12B is 26.3 GB at bf16 and
15.4 GB at official INT8. On 16 GB, use a **GGUF Gemma 4 encoder** (~8.4–9.5 GB) so it can
be swapped out of VRAM before sampling. ComfyUI frees the encoder before the sampling step,
so it does not need to be resident with the transformer.

### 4.1 Colab T4 reality check

The T4 has ~15 GB VRAM **but only ~12.7 GB of host RAM** on Colab, and the offloading that
makes LTX-2.5 fit relies heavily on system RAM (the 16 GB desktop path assumes ~32–48 GB of
RAM). LTX-2.5 on a **Colab T4 is therefore impractical** for anything but the most
aggressive Q2/Q3 GGUF at very low resolution — and the T4 is Turing (sm75), so the
ComfyUI w4a4/w4a8 kernel paths (SM7.5+/SM8.0+) are unproven there. **The T4-viable LTX path
remains LTX-2.3 via GGUF.** Run LTX-2.5 on a larger Colab GPU (L4 24 GB, A100 40/80 GB) or a
local 24 GB+ card; on 24 GB use INT8 or GGUF Q6_K/Q8_0.

## 5. ComfyUI workflows

LTX-2.5 is natively supported in ComfyUI (day-one launch partnership). Built-in templates:

| Goal | Template |
|---|---|
| Text-to-Video | `video_ltx2_5_t2v` |
| Image-to-Video | `video_ltx2_5_i2v` |
| First-Frame / Last-Frame | `video_ltx2_5_flf2v` |
| Audio-to-Video | `LTX-2.5_A2V_Two_Stage_Distilled.json` |
| Text-to-Audio | `LTX-2.5_T2A_Single_Stage_Distilled.json` |
| Advanced two-stage T2V/I2V | `LTX-2.5_T2V_I2V_Two_Stage_Distilled.json` |

Additional editing/VFX graphs (from the LTX ComfyUI nodes repo and docs): V2V IC-LoRA edit,
Inpainting, Outpainting, Union Control (depth/edges/pose), Motion Track, Ingredients,
Tiled Fusion upscale, SDR→HDR, HDR inpaint, HDR I2V, Layout to Render, AlphaGen.

Two-stage flow: Stage 1 samples video + audio at base resolution; the video latent is
upscaled 2× spatially; Stage 2 refines at full resolution; audio is carried through from
Stage 1. Tiled VAE decode keeps peak VRAM down.

For the lower-VRAM (16 GB) path, replace the bf16 files with the GGUF transformer and GGUF
Gemma 4 encoder, keep the stock bf16 VAEs, and use tiled decode.

## 6. Constraints & prompting

- **Frame count must satisfy `num_frames % 8 == 1`** (1, 9, 17, …, 97, 121, …).
- **Width and height must be divisible by 32.**
- Retake requires the source frame count to be `8k + 1` and resolution multiples of 32.
- Describe the **whole scene in one flowing paragraph** — shot type, scene, action,
  characters, camera movement — and describe the **audio** you want (dialogue, SFX, music).
- **T2V/I2V**: write a simple scene idea and let the prompt enhancer expand it.
- **I2V**: describe what happens *next*; do not re-describe what is already visible; anchor
  with phrasing like "use the provided start image as the first frame".
- **Inpainting/outpainting**: describe the **full scene**, not the edit.
- A default negative prompt is applied automatically in the templates.

## 7. Integration with this skill's pipeline

- After the visual-reconstruction and asset-extraction stages, use **I2V** to animate a
  reconstructed still, or **T2V** to generate motion for missing sequences.
- Feed **SAM 2.1 masks** (from the segmentation stage) straight into the LTX **inpainting /
  outpainting** workflows for object replacement or canvas extension.
- Use **Retake** to fix a weak segment instead of re-rendering the whole clip.
- Use **AlphaGen** and the transparent-asset logic together — both produce alpha for
  compositing.
- Use **A2V / T2A** to tie the video branch to the audio branch (ACE-Step / Stable Audio
  Open), or let LTX-2.5 generate synchronised audio natively.
- Use **Refine / Restore / SDR→HDR / Layout to Render** as post-production passes on
  reconstructed footage.

## Sources

Lightricks LTX-2.5 model card, LTX docs (model, capabilities, editing, in/outpainting,
audio-to-video, workflow compatibility), the Lightricks/LTX-2 GitHub repository and
CHANGELOG v1.2.0, ComfyUI LTX-2.5 tutorials and templates, fal.ai and VentureBeat launch
coverage, pIXELsHAM VFX write-up, and community VRAM/GGUF guides (vramready, ltxworkflow,
realrebelai/Abiray/elix3r quant repos).
