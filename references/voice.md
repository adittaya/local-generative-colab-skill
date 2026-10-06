# Voice — Generation & Word-Level Transcription

Two jobs live here:

1. **Voice generation (TTS)** — synthesise speech and clone voices.
2. **Word-level transcription (ASR + alignment)** — turn audio into text with per-word
   timestamps (and, optionally, speaker labels).

The two headline models are **Qwen3-TTS** (voice generation) and the
**Qwen3-ASR + Qwen3-ForcedAligner** pair (word-level transcription). Both are Apache 2.0.

## 1. Voice generation — Qwen3-TTS

Alibaba's Qwen3-TTS family, open-sourced January 2026. It is the most capable open TTS
model of its generation and the recommended default.

- **Sizes:** 1.7B and 0.6B. The 1.7B is peak quality; the 0.6B is the efficiency pick.
- **Variants:** `Base` (3-second rapid voice clone), `CustomVoice` (9 premium timbres),
  `VoiceDesign` (create a voice from a text description).
- **Control:** natural-language voice direction — "speak slowly with a warm, reassuring
  tone" — instead of SSML or phoneme markup.
- **Languages:** 10 (Chinese, English, Japanese, Korean, German, French, Russian,
  Portuguese, Spanish, Italian) plus dialects.
- **Speed:** extreme streaming via a 12 Hz multi-codebook tokenizer and dual-track
  modelling — the first audio packet after a single character.
- **Benchmarks:** SOTA among open models; on a 10-language multilingual set it reports an
  average WER of 1.835% and speaker similarity 0.789, beating MiniMax and ElevenLabs on
  cloning stability; its VoiceDesign beats MiniMax's closed voice-design model.
- **Hardware:** ~6 GB VRAM; CPU works but is slow.
- **Licence:** Apache 2.0.

**Strong alternates:** **Chatterbox** (350M, MIT — the lightest high-quality cloner, 23+
languages, paralinguistic tags like `[laugh]`), **VoxCPM2** (2B, Apache 2.0, 30 languages,
48 kHz, voice design + controllable cloning, ~8 GB), **Kokoro** (82M, CPU-only, English),
**Fish Audio S2 Pro** (4.4B, highest quality, heavier). Avoid **XTTS-v2** for commercial
work — its weights are non-commercial (Coqui Public Model Licence).

## 2. Word-level transcription — Qwen3-ASR + Qwen3-ForcedAligner

The Qwen3-ASR family (Apache 2.0) pairs a recognition model with a dedicated
forced-alignment model — exactly the "audio → word-level transcript" stack.

- **Qwen3-ASR-1.7B / 0.6B** — all-in-one ASR with language identification for **52
  languages and dialects**. The 1.7B is state-of-the-art among open ASR models and
  competitive with the strongest proprietary APIs; the 0.6B is the accuracy/efficiency
  pick (time-to-first-token as low as **92 ms**; transcribes 2,000 s of speech in 1 s at
  concurrency 128).
- **Qwen3-ForcedAligner-0.6B** — a non-autoregressive, LLM-based forced aligner that
  predicts **word-level (and sentence/paragraph) start/end timestamps** given audio and its
  transcript, across 11 languages, in under five minutes. It reports a **67–77% relative
  reduction in timestamp shift** versus other forced-alignment methods.

**Strong alternates:**

- **IBM Granite-Speech-4.1-2B-Plus** (Apache 2.0) — produces **word-level timestamps and
  speaker diarization in a single pass** (plus speaker-attributed ASR), with a strong
  English word-timestamp accuracy (~38.8 ms average AAS). Best when you want timing *and*
  "who spoke when" without a multi-model pipeline. Covers English, French, German, Spanish,
  Portuguese.
- **Whisper Large V3 + WhisperX** (MIT) — the classic word-level pipeline: Whisper for
  transcription across 99+ languages, then wav2vec2 forced alignment for word timestamps.
  Most ecosystem, widest language coverage; accuracy now trails the newer models.
- **NVIDIA Parakeet TDT** — maximum throughput; **NVIDIA Canary-Qwen 2.5B** — top benchmark
  WER (~5.63%).

## 3. Pipeline

1. **Transcribe** — Qwen3-ASR → text + language ID.
2. **Align** — Qwen3-ForcedAligner → per-word start/end timestamps.
3. **(Optional) Diarize** — speaker labels per word/segment.
4. **Emit** — SRT / VTT / TXT / JSON.

For voice generation: **reference clip → clone → synthesise** (or VoiceDesign from a text
description), then post-process (normalise, de-ess, match loudness to the mix).

## 4. Integration with the pipeline

- **Word-level timestamps** drive subtitles, **lip-sync** (LTX-2.5 Dub-It, Wan 2.2 S2V,
  MOVA), lyric alignment for ACE-Step music, and searchable/clickable transcripts.
- **TTS output** supplies narration and dialogue for the reconstructed video, dubbing, and
  avatar/character voices.
- **Round trips:** ASR → LLM → TTS (a voice agent); or ASR → translate → TTS → lip-sync
  (localisation).
- Feed a **word-level transcript + its audio** into VoxCPM2's "Ultimate Cloning" for
  audio-continuation cloning that reproduces every vocal nuance.

## 5. Hardware note

On a Colab T4 (~15 GB VRAM), Qwen3-TTS 1.7B (~6 GB) and Qwen3-ASR 1.7B both fit — run them
**sequentially** with the other heavy stages, never concurrently. The 0.6B variants are the
safe defaults when VRAM is tight.

## Sources

Qwen3-TTS (Alibaba Cloud community post, Hugging Face), Qwen3-ASR technical report
(arXiv 2601.21337), IBM Granite-Speech-4.1-2B-Plus model card, WhisperX, NVIDIA Canary /
Parakeet (NeMo), VoxCPM2, Chatterbox, Kokoro and community TTS/ASR round-ups (2026).
