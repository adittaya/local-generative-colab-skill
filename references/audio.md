# Audio Reconstruction & Generation

The audio branch has two jobs: **understand/reconstruct** the source audio, and
**generate** the music, SFX and ambience the reconstructed scene needs.

## Models

### Music — ACE-Step 1.5

An open-source **music foundation model** with a hybrid architecture: a language model acts
as the planner (turning a short query into a full song blueprint — structure, lyrics,
captions) while a Diffusion Transformer renders the audio.

- **Sizes:** 2B DiT (`base` / `sft` / `turbo`) and XL 4B DiT (`xl-base` / `xl-sft` /
  `xl-turbo`).
- **VRAM:** the 2B model runs in **under ~4 GB**; the XL 4B needs **≥12 GB with offload**
  (~20 GB without). A T4 fits the 2B comfortably and the XL with offload.
- **Speed:** a full song in **under 2 s on an A100**, under ~10 s on an RTX 3090.
- **Duration:** ~10 s to ~600 s (10 minutes).
- **Capabilities:** text-to-music, cover generation, repainting, vocal-to-BGM conversion,
  extract / lego / complete, reference-audio conditioning; 50+ languages; LoRA from a few
  songs.
- **Licence:** MIT (per the model card); trained on licensed / royalty-free / synthetic data
  and cleared for commercial use.

### SFX & ambience — Stable Audio Open 1.5

Optimised for **sound design and textural audio** rather than songs.

- ~1.1B parameters, ~12 GB VRAM, up to ~47 s output at 44.1 kHz.
- Best for **sound effects, foley, textures and ambience** — not full tracks.
- Licence: Stability AI Community Licence (commercial use under a revenue threshold — check
  the current terms).

### Synchronised AV audio

Where audio must stay locked to picture, generate it **jointly with the video** via
**LTX-2.5** (audio-to-video, text-to-audio) or **MOVA** — rather than a separate music/SFX
pass. See `video-generation.md`.

## Pipeline

1. **Analyse** the source audio — tempo, key, mood, dialogue, loudness.
2. **Decide** per element: reconstruct from source, or generate fresh.
3. **Generate** music (ACE-Step 1.5) and SFX/ambience (Stable Audio Open 1.5).
4. **Align & mix** — normalise loudness, match the generated audio to the video timing.
5. **Write metadata** and render previews.

## Outputs

- `WAV` / `FLAC` masters per asset.
- `metadata JSON` — model, version, prompt, duration, sample rate, seed, generation time.
- `preview` clips.

## Hardware note

On a Colab T4, run the audio models **sequentially** with the video/3D stages rather than
concurrently — the T4 has ~15 GB VRAM and limited host RAM, so one heavy model at a time.
The ACE-Step 2B model is the lightest of the audio set and the safest default.

## Sources

ACE-Step 1.5 repository, model card and paper ("Pushing the Boundaries of Open-Source Music
Generation", arXiv 2602.00744); Stable Audio Open 1.5 model card; community GPU-deployment
guides (Spheron).
