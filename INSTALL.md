# Install — local-generative-colab-skill

Copy the block below and paste it into your agent to install and run the skill.

```text
Install and run the "local-generative-colab-skill".

Repository: https://github.com/adittaya/local-generative-colab-skill

1. Read SKILL.md at the repository root. It defines your role, the non-negotiable rules,
   the pipeline stages and the reference index.
2. Load the reference files from references/ as each stage needs them:
   - model-discovery.md
   - architecture-and-cli.md
   - remote-compute.md
   - resource-discipline.md
   - visual-reconstruction.md
   - visual-understanding-and-segmentation.md
   - asset-extraction-and-generation.md
   - 3d-generation.md
   - video-generation.md
   - audio.md
   - voice.md
   - model-selection.md
3. Operate as the skill describes: you are the CONTROLLER on the user's local Linux
   machine, and ALL work runs on a REMOTE cloud GPU - Google Colab (1x T4,
   ~16 GB) or Kaggle (2x T4, ~32 GB) - through the Colab CLI or the Kaggle CLI. Run
   everything there, heavy and light (inference, downloads, packaging, editing, assembling);
   the local machine only saves files, runs the controller scripts and collects outputs.
   Choose the backend per task (Kaggle for heavier/longer jobs, Colab for interactive iteration).
   Inspect the installed CLI and adapt to its real syntax, verify the remote GPU, then
   actually build / execute / monitor / debug / resume / complete / package / return the
   whole project.
   Treat BOTH remote filesystems as temporary scratch; the local machine's home directory is
   the source of truth - stage inputs out and pull outputs and checkpoints back to local.
4. Do not merely explain the workflow, do not give a tutorial, do not tell the user to do
   anything by hand, and do not stop after generating scripts. Execute.
5. Priority order throughout: Quality > Fidelity > Editability > Speed.
6. Never waste the remote GPU: run ONE task at a time, and after each task's outputs are
   pulled back to local, unload the model, free the GPU, and stop the session if nothing
   else needs it. There is no working time limit - prioritise quality - but never leave a
   session idle or a second heavy model resident. The local machine only saves files, runs
   the controller scripts and collects outputs.

The skill covers six branches: visual reconstruction; editable asset extraction (SAM 2.1
Large + BiRefNet); 3D asset generation (Hunyuan3D 2.1); audio reconstruction/generation
(ACE-Step 1.5, Stable Audio Open 1.5); voice generation & word-level transcription
(Qwen3-TTS, Qwen3-ASR + Qwen3-ForcedAligner); and video generation & regeneration (LTX-2.5).
Use references/model-selection.md to choose the right expert model for each task, and
docs/generation-times.html for local video render-time expectations.

For every task, research and select the current best specialist model rather than
defaulting to one all-rounder (see references/model-discovery.md). Never assume the models
named in the skill are still the best.

Begin by reading SKILL.md, then present the plan as todos and start executing.
```

Repo: <https://github.com/adittaya/local-generative-colab-skill>
