# local-generative-colab-skill

A skill that drives a heavy **generative-media pipeline** from a local Linux machine while
all computation runs on a **remote Google Colab GPU** (NVIDIA T4, ~16 GB) via the Colab CLI.

It covers five coupled branches:

- **Visual reconstruction** — reconstruct high-quality imagery from source frames using a
  top-tier generative model, conditioned on the source (not a plain pixel upscaler).
- **Editable asset extraction** — decompose a scene into maximally editable transparent 2D
  assets using SAM 2.1 Large for segmentation and BiRefNet for refinement.
- **3D asset generation** — turn genuinely volumetric elements into editable 3D assets with
  Hunyuan3D 2.1.
- **Audio reconstruction / generation** — music, SFX and ambience for the reconstructed scene.
- **Video generation & regeneration** — generate or regenerate motion with **LTX-2.5**
  (Lightricks' open-weights audio-video world model): T2V/I2V/A2V, native multishot and
  synchronised audio, plus Retake, extend, inpainting/outpainting, IC-LoRA transforms,
  upscale/restore and SDR→HDR.

## Layout

```
SKILL.md                                     Skill entry point (frontmatter + workflow)
README.md                                    This file
references/
  architecture-and-cli.md                    Architecture, remote-GPU rules, Colab CLI, project dirs
  visual-reconstruction.md                   Objective, image-generation quality stage, models
  visual-understanding-and-segmentation.md   Understanding, editability, SAM 2.1, BiRefNet
  asset-extraction-and-generation.md         Classification, extraction vs generation, text, logos
  3d-generation.md                           Hunyuan3D 2.1 3D asset generation
  video-generation.md                        LTX-2.5 video generation & regeneration
```

## Core idea

The local Linux machine is **only the controller**. Every heavy AI workload — image
generation, 3D generation, large-model segmentation, audio generation, reconstruction —
runs on the remote Colab GPU. The controller inspects the installed Colab CLI, adapts to its
actual syntax, verifies the remote GPU, and builds / executes / monitors / debugs / resumes /
packages / returns the whole project without asking the user to do anything by hand.

Priority order throughout: **Quality > Fidelity > Editability > Speed**.

## Using it

Point your agent at `SKILL.md` and follow the workflow, pulling detail from `references/` as
each stage needs it. The skill is written to be executed end to end against a live Colab
runtime, not read as a tutorial.

## Note on source material

The instructions were assembled from a master production brief. The brief was truncated
after section 28 (the 3D-generation branch); the detailed audio-pipeline sections referenced
in the architecture diagram were not present in the source and are not included here. Those
sections can be appended to `references/3d-generation.md` (or a new `references/audio.md`)
once available.
