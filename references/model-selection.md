# Model Selection — Use Case → Expert Model

One all-rounder is rarely optimal. This map pairs each concrete task with the specialist
model that does it best, so the pipeline can swap in the right tool per job instead of
forcing one model to do everything.

## Video generation

| Use case | Expert model | Why it wins |
|---|---|---|
| Best overall T2V / I2V quality | **Wan 2.2 (A14B)** | Quality leader on motion realism, anatomy and face/subject consistency; Apache 2.0 |
| Fast high-quality T2V/I2V **with audio** | **LTX-2.5 (distilled)** | ~5.7× faster than Wan 2.2; synchronised audio in one pass |
| Strongest open weights + audio | **MiniMax H3** | 33B; #4 with audio / #2 without; 32 kHz stereo, 11-language dialogue (licence exclusions apply) |
| Long / continuous video | **LongCat-Video** | Native video-continuation; minutes-long without drift |
| Real-time / interactive | **Helios** | ~19.5 FPS on an H100; minute-scale, T2V/I2V/V2V/interactive |
| Video **with transparency (RGBA)** | **Wan-Alpha** | Generates a native alpha channel — semi-transparent objects, glows, hair |
| Strict prompt adherence | **CogVideoX** | Best semantic accuracy on structured, multi-clause prompts |
| Low-VRAM / consumer GPU | **Wan 2.1 1.3B** or **Wan 2.2 TI2V-5B** | Runs from ~8 GB |
| Cinematic faces on one consumer GPU | **HunyuanVideo 1.5** | 8.3B, ~14 GB with FP8 + offload |

## Talking heads, avatars & lip-sync

| Use case | Expert model | Why it wins |
|---|---|---|
| Speech-to-video (talking head) | **Wan 2.2 S2V** | Image + audio → talking video, with pose-driven variants |
| Multilingual lip-sync | **MOVA** | State-of-the-art lip-sync; synchronised audio-video in one pass |
| Audio-driven character animation | **LongCat-Video-Avatar-1.5** | Whisper encoder, stylised domains (anime, animals) |
| Character animation / replacement | **Wan 2.2 Animate** | Mimic motion from a reference, or swap a character in |

## Video regeneration & editing

| Use case | Expert model | Why it wins |
|---|---|---|
| Regenerate a segment (Retake) | **LTX-2.5** | Rewrite a time window; outside stays bit-identical |
| Extend a clip | **LTX-2.5** | Continue from the last frame, no seams |
| Inpainting / object replacement | **LTX-2.5** (In-Outpainting IC-LoRA) | Mask-aware, blends at mask boundaries |
| Outpainting / canvas extension | **LTX-2.5** | Extends sides/top/bottom consistently |
| Style / identity transform (V2V) | **LTX-2.5** (V2V IC-LoRA) | Deblur, colorization, day-to-night, and more |
| Rewrite dialogue, keep identity | **LTX-2.5 Dub-It** | Rephrases while matching speaker + lip movement |
| Transition between keyframes | **LTX-2.5 KeyframeInterpolation** | Generates the in-between motion |
| Structural control (depth/pose/edges) | **LTX-2.5 Union Control** | One checkpoint accepts all three signals |
| Camera-move control | **LTX camera-control LoRAs** / HunyuanVideo | Directed dolly/jib/pan behaviour |

## Post-production & VFX

| Use case | Expert model | Why it wins |
|---|---|---|
| Upscale to HD / 4K / 8K | **LTX Tiled-Fusion upscale** (or RealESRGAN / SeedVR) | Generative re-render, native 4K/8K canvases |
| Restore archival / degraded footage | **LTX Restore** | Revives colour and detail |
| Refine / deblur | **LTX Refine** + deblur IC-LoRA | Rebuilds fine detail in blurry footage |
| SDR → HDR (EXR) | **LTX SDR-to-HDR** | 16-bit EXR in/out for grading |
| Alpha mattes for compositing | **LTX AlphaGen** (beta) / **BiRefNet** | Frame-aligned mattes incl. hair and smoke |
| 3D render / blocking → finished shot | **LTX Layout-to-Render** | Follows camera, composition, object placement |

## Image — this skill's reconstruction branch

| Use case | Expert model | Why it wins |
|---|---|---|
| High-quality reconstruction / edit | **FLUX.2**, **Qwen-Image-Edit**, **FLUX Kontext** | Reference-conditioned reconstruction |
| Fast text-to-image | **Z-Image-Turbo** | Few-step generation |
| Text rendering in image | **Qwen-Image** | Strong typography |

## Segmentation & matting

| Use case | Expert model | Why it wins |
|---|---|---|
| Pixel-level segmentation | **SAM 2.1 Large** | Primary masking engine |
| Foreground / edge refinement | **BiRefNet** | Hair, thin structures, soft edges |

## 3D

| Use case | Expert model | Why it wins |
|---|---|---|
| Image → editable 3D asset | **Hunyuan3D 2.1** | Volumetric elements to editable 3D |

## Audio

| Use case | Expert model | Why it wins |
|---|---|---|
| Music | **ACE-Step 1.5** | Full-track music generation |
| SFX / ambience | **Stable Audio Open** | Sound effects and atmosphere |
| Synchronised AV audio | **LTX-2.5** / **MOVA** | Audio generated jointly with video |

## How to choose

1. Name the **task** (generate / regenerate / edit / restore / control).
2. Match it to the row above; take the specialist, not the generalist.
3. Check **hardware**: quantised paths for 16–24 GB, full precision for 48 GB+.
4. Check **licence** for commercial use (Apache 2.0 is cleanest; LTX is free under ~$10M
   ARR; MiniMax H3 and some others exclude specific regions).
5. Fall back to the all-rounder (**LTX-2.5**) only when no specialist fits or when audio
   must be generated jointly with video.

## Sources

Wan-Video, Lightricks LTX-2.5, MiniMax H3, LongCat-Video, Helios (PKU), Wan-Alpha (WeChatCV),
MOVA (OpenMOSS), HunyuanVideo/Hunyuan3D (Tencent), CogVideoX (Zhipu), Lightricks LTX-2 repo
and docs, and community comparison benchmarks.
