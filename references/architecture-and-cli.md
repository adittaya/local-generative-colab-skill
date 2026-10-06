## 1. FUNDAMENTAL ARCHITECTURE

> **How to use this module.** Load this **first, in every run**. It defines how to reach and verify the remote Colab GPU; follow it before any heavy stage. Before locking a model, refresh the pick via `references/model-discovery.md`.

MY LOCAL LINUX COMPUTER
        |
        | Google Colab CLI
        v
REMOTE GOOGLE COLAB
        |
        v
NVIDIA T4 GPU
        |
        +------------------------------------------------+
        |                                                |
        v                                                v
VISUAL PIPELINE                                  AUDIO PIPELINE
        |                                                |
        v                                                |
HIGH-QUALITY IMAGE                           AUDIO ANALYSIS
RECONSTRUCTION                                      |
        |                                   +----------+----------+
        v                                   |                     |
VISUAL ANALYSIS                              v                     v
        |                              ACE-Step 1.5       Stable Audio Open
        v                                  MUSIC             SFX / AMBIENCE
SAM 2.1 LARGE
        |
        v
BIREFNET
        |
        v
TRANSPARENT 2D ASSETS
        |
        +-------------------------+
        |                         |
        v                         v
2D RECONSTRUCTION           3D ASSET CANDIDATES
                                  |
                                  v
                           HUNYUAN3D 2.1
                                  |
                                  v
                             3D ASSETS
        |
        v
GENERATE MISSING /
NON-EXTRACTABLE ASSETS
        |
        v
LAYER RECONSTRUCTION
        |
        v
QUALITY CONTROL
        |
        v
FINAL ASSET LIBRARY
        |
        v
ZIP RESULTS
        |
        v
RETURN TO LOCAL MACHINE

---

## 2. ABSOLUTE ARCHITECTURE RULE

The local Linux machine is ONLY the controller.

All heavy AI computation MUST happen on Google Colab.

DO NOT perform heavy inference on the local computer.

DO NOT use the local CPU/GPU for:

- image generation
- 3D generation
- large-model segmentation
- audio generation
- heavy reconstruction
- large-model inference

DO NOT ask me to:

- manually open a Colab notebook
- manually authenticate Google
- manually upload files
- copy/paste code into Colab
- manually execute Python in Colab
- manually install heavy AI dependencies in Colab

Google Colab is already authenticated/configured through the
Google Colab CLI.

Use the CLI from the local terminal to control the remote environment.

---

## 3. GOOGLE COLAB CLI

First inspect the currently installed CLI.

Use commands such as:

colab --help
colab --version
colab sessions

BUT:

DO NOT blindly assume that a particular CLI syntax is valid.

The installed CLI version is authoritative.

Inspect its help and adapt to the actual available commands.

If the syntax differs from older examples:

1. inspect the installed version
2. inspect --help
3. determine correct commands
4. adapt automatically
5. continue

Preferred session name:

assetjob

If assetjob already exists:

- inspect it
- determine whether it is active
- reuse it if useful
- do not destroy useful existing work

Create a dedicated remote job/session when necessary.

---

## 4. REMOTE GPU REQUIREMENT

Expected GPU:

NVIDIA T4

Expected VRAM:

approximately 16 GB

Inside the remote Colab runtime verify:

nvidia-smi

Also verify:

- CUDA availability
- PyTorch CUDA availability
- GPU name
- GPU VRAM
- Python version
- CUDA version
- PyTorch version

The actual heavy computation MUST run on the remote GPU.

If a GPU is unavailable:

DO NOT silently use the local machine.

Clearly report the problem and stop heavy GPU inference.

---

## 5. INPUT

I will provide source material from the local machine.

VISUAL INPUT MAY INCLUDE:

PNG
JPG
JPEG
WEBP
TIFF
BMP
video frames
image sequences
reference images
ZIP archives

AUDIO INPUT MAY INCLUDE:

WAV
MP3
FLAC
M4A
AAC
OGG

The input may be supplied as a ZIP.

Automatically locate the supplied input on the local machine.

Do not make me manually specify paths unless the input genuinely
cannot be found.

Transfer/provide the input to the remote Colab runtime using the
available Colab CLI workflow.

DO NOT require manual browser upload.

Preserve original files unchanged.

---

## 6. REMOTE PROJECT DIRECTORY

Create:

/content/project/

with:

/content/project/input/
/content/project/original/
/content/project/work/
/content/project/reconstructed/
/content/project/assets/
/content/project/assets/visual/
/content/project/assets/3d/
/content/project/assets/audio/
/content/project/masks/
/content/project/previews/
/content/project/metadata/
/content/project/ocr/
/content/project/generation/
/content/project/generation/prompts/
/content/project/generation/references/
/content/project/audio_generation/
/content/project/audio_generation/music/
/content/project/audio_generation/sfx/
/content/project/audio_generation/ambience/
/content/project/reconstruction/
/content/project/logs/
/content/project/checkpoints/
/content/project/output/

Keep all processing deterministic and organized.
