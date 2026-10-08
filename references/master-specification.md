# Master Specification — Specialist Generative-Media System

> **How to use this module.** This is the **top-level system spec**. Load it first, with
> `model-discovery.md` (how to choose) and `model-selection.md` (the live picks). It defines
> the purpose, the domain taxonomy, the architecture and the routers.

## 1. Purpose

Build a modular generative-media system in which **no single large model is expected to do
every task**. For each request:

1. Understand the task.
2. Classify the exact modality and objective.
3. Determine hardware constraints.
4. Research/select the best **specialist** model.
5. Launch **only that model** on the remote GPU.
6. Produce the requested asset.
7. Run optional refinement / post-processing.
8. Save reproducible metadata.
9. **Release the remote GPU when finished.**

**Fundamental rule: SPECIALIST > GENERALIST.** A general-purpose model is only a fallback
when no suitable specialist exists.

## 2. Supported media domains

**Image** — generation, reconstruction, editing, image-to-image, visual understanding,
segmentation, matting/background removal, asset extraction, asset generation, typography/text
reconstruction, logo reconstruction.

**3D** — asset generation, reconstruction.

**Video** — text-to-video, image-to-video, audio-to-video, video-to-video, regeneration,
extension, interpolation, inpainting, outpainting, character animation, talking-head, lip
sync, avatar generation, camera-motion control, restoration, enhancement, upscaling, SDR→HDR,
alpha/matte generation.

**Audio** — music generation, sound-effect generation, ambient sound, voice synthesis, voice
cloning, multilingual voice, speech recognition, forced alignment, speaker diarization,
subtitle generation, audio/video synchronization.

**Assembly** — final compositing, media assembly, format conversion, quality-control analysis.

## 3. System architecture

```
USER REQUEST → TASK UNDERSTANDING → TASK CLASSIFICATION
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      IMAGE          VIDEO          AUDIO
        ▼              ▼              ▼
       3D           CHARACTER        VOICE
        └──────────────┼──────────────┘
                       ▼
              MODEL DISCOVERY  →  HARDWARE / VRAM CHECK  →  LICENSE VALIDATION
                       ▼
              PRIMARY + FALLBACK  →  REMOTE GPU JOB  →  QUALITY CONTROL
                       ▼
              OPTIONAL REFINEMENT  →  FINAL ASSET  →  MANIFEST + METADATA  →  RETURN / ARCHIVE
```

## 4. Model discovery engine

Never hard-code the assumption that a particular model is permanently best. Evaluate each
candidate on: task-specific quality, release date, version, parameter count, VRAM/RAM,
quantization and CPU-offload support, FP16/BF16, speed, resolution, temporal consistency,
identity consistency, prompt adherence, ecosystem (ComfyUI/Diffusers), LoRA and control
support, fine-tuning, licence (commercial and geographic restrictions), reproducibility and
weight availability. Record per selection: `model_name, model_version, release_date,
parameter_count, license, source_url, task, why_selected, hardware_path, quantization,
fallback, validation_date`. See `model-discovery.md` — the selection table is a **snapshot**,
refreshed before production use.

## 5. Hardware router

Before loading any model, check GPU → VRAM → RAM → CUDA → PyTorch → model compatibility.

| VRAM | Strategy |
|---|---|
| ~8 GB | lightweight / quantized specialists |
| ~12–16 GB | T4-class optimised models + quantization + offloading |
| ~24 GB | larger specialist models |
| ~32 GB | larger models, or two-GPU workflows |
| 48 GB+ | high-quality unquantized / large specialist paths |

**Never assume a model fits merely because it is open-source.**

## 6. Backend router

- **Google Colab** — interactive iteration, quick tests, debugging, rapid prompt
  experimentation, smaller jobs.
- **Kaggle** — longer jobs, heavier workloads, batching, larger RAM/VRAM configs.

Run both in parallel where it helps; see `remote-compute.md`, `resource-discipline.md` and
`quota-and-accounts.md`.

## 7. Domain → specialist route

The live, refreshed picks live in `model-selection.md`. The canonical route:

| Domain | Specialist |
|---|---|
| Text → image | FLUX.2 / Qwen-Image (text) / Z-Image-Turbo (fast) / FLUX Kontext (edit) |
| Image reconstruction | reference-conditioned generative reconstruction (not super-resolution) |
| Image editing | specialist image-edit model (source + mask + reference) |
| Visual understanding | vision-language model |
| Segmentation | **SAM 2.1 Large** |
| Matte refinement | **BiRefNet** |
| Text reconstruction | OCR + typography analysis (never trust generation for text) |
| Logo handling | exact extraction → transparent reconstruction → manual-review flag |
| Image → 3D | **Hunyuan3D 2.1** |
| Text → video | Wan 2.2 A14B (quality) / LTX-2.5 (speed + audio) |
| Image → video | dedicated I2V model |
| Audio → video | audio-conditioned video model |
| Talking head | Wan 2.2 S2V |
| Lip sync | MOVA (multilingual) |
| Character animation | Wan 2.2 Animate / LongCat-Video-Avatar |
| Video regeneration / edit | **LTX-2.5** (Retake, Extend, in/outpaint, keyframe, control) |
| Camera control | LTX camera-control LoRAs / HunyuanVideo |
| Video upscale | generative upscaler / Real-ESRGAN / SeedVR / LTX tiled-fusion |
| Video restoration | dedicated restoration chain (denoise → deblur → colour → detail → temporal) |
| SDR → HDR | generative HDR reconstruction (16-bit intermediate) |
| Alpha / matte generation | frame-aligned matte model (people, hair, smoke, particles) |
| Music | **ACE-Step 1.5** |
| Sound effects | **Stable Audio Open** |
| Ambience | ambience layer (separate from music and dialogue) |
| Synchronised audio+video | **LTX-2.5** / MOVA |
| Text → speech | **Qwen3-TTS** |
| Voice cloning | Qwen3-TTS / IndexTTS-2 (see `voice.md`) |
| Indic voices | **IndicF5** |
| Speech-to-text | **Qwen3-ASR** / Whisper Large V3 / WhisperX |
| Word-level alignment | **Qwen3-ASR + Qwen3-ForcedAligner** |
| Speaker diarization | Granite-Speech / WhisperX-style ecosystems |
| Subtitles | ASR → word alignment → segmentation → SRT/VTT/ASS |

## 8. Documentary production pipeline

```
RESEARCH / SCRIPT → SHOT LIST → REFERENCE IMAGES → IMAGE RECONSTRUCTION / GENERATION
→ IMAGE → VIDEO → CHARACTER / TALKING HEAD → VOICEOVER → MUSIC → SFX → AMBIENCE
→ SUBTITLES → COMPOSITING → COLOR / HDR → UPSCALE → MASTER
```

## 9. Shot specification

Every shot carries a structured spec so the whole piece is reproducible:

```
shot_id, duration, aspect_ratio, resolution, scene, subject, action, environment,
camera, lens, lighting, style, motion, dialogue, voice, music, sfx, ambience,
negative_prompt, seed, model, model_version
```

## Sources

Master specification v1.0 (specialist-model architecture, 2026); cross-references to
`model-discovery.md`, `model-selection.md`, `remote-compute.md` and the domain modules.
