# Model Selection — Use Case → Expert Model

One all-rounder is rarely optimal. This map pairs each concrete task with the specialist
model that does it best, so the pipeline can swap in the right tool per job instead of
forcing one model to do everything.

The **Evidence** column is honest about how well-supported each pick is:

- **High** — reproduced across multiple independent benchmarks or official docs.
- **Medium** — vendor benchmarks and/or consistent community reports, but few independent runs.
- **Thin** — a single source or a close call; verify before committing.

## Video generation

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Best overall T2V / I2V quality | **Wan 2.2 (A14B)** | Quality leader on motion realism, anatomy and face/subject consistency; Apache 2.0 | High |
| Fast high-quality T2V/I2V **with audio** | **LTX-2.5 (distilled)** | ~5.7× faster than Wan 2.2; synchronised audio in one pass | High |
| Strongest open weights + audio | **MiniMax H3** | 33B; #4 with audio / #2 without; 32 kHz stereo, 11-language dialogue (licence exclusions apply) | Medium |
| Long / continuous video | **LongCat-Video** | Native video-continuation; minutes-long without drift | Medium |
| Real-time / interactive | **Helios** | ~19.5 FPS on an H100; minute-scale, T2V/I2V/V2V/interactive | Medium |
| Video **with transparency (RGBA)** | **Wan-Alpha** | Generates a native alpha channel — semi-transparent objects, glows, hair | Medium |
| Strict prompt adherence | **CogVideoX** | Best semantic accuracy on structured, multi-clause prompts | Medium |
| Low-VRAM / consumer GPU | **Wan 2.1 1.3B** or **Wan 2.2 TI2V-5B** | Runs from ~8 GB | High |
| Cinematic faces on one consumer GPU | **HunyuanVideo 1.5** | 8.3B, ~14 GB with FP8 + offload | High |

## Talking heads, avatars & lip-sync

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Speech-to-video (talking head) | **Wan 2.2 S2V** | Image + audio → talking video, with pose-driven variants | Medium |
| Multilingual lip-sync | **MOVA** | State-of-the-art lip-sync; synchronised audio-video in one pass | Medium |
| Audio-driven character animation | **LongCat-Video-Avatar-1.5** | Whisper encoder, stylised domains (anime, animals) | Thin |
| Character animation / replacement | **Wan 2.2 Animate** | Mimic motion from a reference, or swap a character in | Medium |

## Video regeneration & editing

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Regenerate a segment (Retake) | **LTX-2.5** | Rewrite a time window; outside stays bit-identical | High |
| Extend a clip | **LTX-2.5** | Continue from the last frame, no seams | High |
| Inpainting / object replacement | **LTX-2.5** (In-Outpainting IC-LoRA) | Mask-aware, blends at mask boundaries | High |
| Outpainting / canvas extension | **LTX-2.5** | Extends sides/top/bottom consistently | High |
| Style / identity transform (V2V) | **LTX-2.5** (V2V IC-LoRA) | Deblur, colorization, day-to-night, and more | Medium |
| Rewrite dialogue, keep identity | **LTX-2.5 Dub-It** | Rephrases while matching speaker + lip movement | Medium |
| Transition between keyframes | **LTX-2.5 KeyframeInterpolation** | Generates the in-between motion | High |
| Structural control (depth/pose/edges) | **LTX-2.5 Union Control** | One checkpoint accepts all three signals | Medium |
| Camera-move control | **LTX camera-control LoRAs** / HunyuanVideo | Directed dolly/jib/pan behaviour | Medium |

## Post-production & VFX

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Upscale to HD / 4K / 8K | **LTX Tiled-Fusion upscale** (or RealESRGAN / SeedVR) | Generative re-render, native 4K/8K canvases | Medium |
| Restore archival / degraded footage | **LTX Restore** | Revives colour and detail | Medium |
| Refine / deblur | **LTX Refine** + deblur IC-LoRA | Rebuilds fine detail in blurry footage | Medium |
| SDR → HDR (EXR) | **LTX SDR-to-HDR** | 16-bit EXR in/out for grading | Medium |
| Alpha mattes for compositing | **LTX AlphaGen** (beta) / **BiRefNet** | Frame-aligned mattes incl. hair and smoke | Thin |
| 3D render / blocking → finished shot | **LTX Layout-to-Render** | Follows camera, composition, object placement | Medium |

## Image — this skill's reconstruction branch

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| High-quality reconstruction / edit | **FLUX.2**, **Qwen-Image-Edit**, **FLUX Kontext** | Reference-conditioned reconstruction | High |
| Fast text-to-image | **Z-Image-Turbo** | Few-step generation | Medium |
| Text rendering in image | **Qwen-Image** | Strong typography | Medium |

## Segmentation & matting

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Pixel-level segmentation | **SAM 2.1 Large** | Primary masking engine | High |
| Foreground / edge refinement | **BiRefNet** | Hair, thin structures, soft edges | High |

## 3D

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Image → editable 3D asset | **Hunyuan3D 2.1** | Volumetric elements to editable 3D | High |

## Audio

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Music | **ACE-Step 1.5** | Full-track music generation | Medium |
| SFX / ambience | **Stable Audio Open** | Sound effects and atmosphere | Medium |
| Synchronised AV audio | **LTX-2.5** / **MOVA** | Audio generated jointly with video | High |

## Voice

| Use case | Expert model | Why it wins | Evidence |
|---|---|---|---|
| Voice generation / cloning (TTS) | **Qwen3-TTS** (1.7B) | SOTA open TTS; 3-second clone; natural-language voice control; Apache 2.0 | High |
| Lightweight voice cloning | **Chatterbox** (350M) | MIT; best clone quality per size; 23+ languages; paralinguistic tags | Medium |
| Multilingual cloning + voice design | **VoxCPM2** (2B) | 30 languages, 48 kHz, controllable cloning; Apache 2.0 | Medium |
| Word-level transcription (ASR) | **Qwen3-ASR + Qwen3-ForcedAligner** | 52 languages; dedicated word-level forced alignment; Apache 2.0 | High |
| Timestamps + speaker diarisation in one pass | **IBM Granite-Speech-4.1-2B-Plus** | Native word timestamps and speaker labels, no extra pipeline | Medium |
| Widest language coverage transcription | **Whisper Large V3 + WhisperX** | 99+ languages; largest ecosystem; MIT | High |

## How to choose

1. Name the **task** (generate / regenerate / edit / restore / control).
2. Match it to the row above; take the specialist, not the generalist.
3. Weigh the **Evidence** column — prefer High rows; prototype before betting on Thin ones.
4. Check **hardware**: quantised paths for 16–24 GB, full precision for 48 GB+.
5. Check **licence** for commercial use (Apache 2.0 is cleanest; LTX is free under ~$10M
   ARR; MiniMax H3 and some others exclude specific regions).
6. Fall back to the all-rounder (**LTX-2.5**) only when no specialist fits or when audio
   must be generated jointly with video.

## Sources

Wan-Video, Lightricks LTX-2.5, MiniMax H3, LongCat-Video, Helios (PKU), Wan-Alpha (WeChatCV),
MOVA (OpenMOSS), HunyuanVideo/Hunyuan3D (Tencent), CogVideoX (Zhipu), Lightricks LTX-2 repo
and docs, and community comparison benchmarks. Benchmark coverage is uneven across models —
treat Medium/Thin rows as directions to verify, not settled facts.
