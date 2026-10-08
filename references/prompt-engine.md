# Prompt Engine & Negative-Prompt Engine

> **How to use this module.** Load it whenever a model is about to be called. It defines how
> to turn a request into a **task-specific, model-aware** prompt — and how to steer away from
> the failure modes.

## 1. Prompt engine

**Do not send the user's raw sentence to every model.** Convert it into a task-specific,
**model-aware** prompt. A Wan prompt and an audio-model prompt should not be identical.

### Text → video structure

```
SUBJECT + ACTION + ENVIRONMENT + CAMERA + LIGHTING + MATERIAL
+ MOTION + TEMPORAL BEHAVIOR + STYLE + QUALITY CONSTRAINTS
```

Example:

> A lone astronaut walking slowly across a vast red desert on Mars, dust drifting around the
> boots, distant mountains visible through thin atmospheric haze, subtle fabric movement,
> realistic human walking cycle, slow cinematic dolly-in camera, natural parallax, warm
> low-angle sunlight, physically plausible shadows, realistic materials, restrained motion,
> stable geometry, cinematic documentary photography.

### Image → video motion spec

> Subject remains visually consistent with the reference image. Subtle natural breathing and
> head movement. Hair moves gently in the wind. Background remains stable. Slow cinematic
> camera push-in. No deformation of facial identity. No changes to clothing design.

### Camera vs subject (keep them independent)

```
CAMERA:      slow dolly forward
SUBJECT:     walking toward camera
ENVIRONMENT: wind moving vegetation
BACKGROUND:  stable
LENS:        cinematic shallow depth of field
```

### Music prompt

> cinematic documentary score, slow orchestral build, restrained percussion, warm strings,
> subtle ambient texture, emotional but not melodramatic, gradual crescendo, space for
> narration, no dominant lead melody.

### Routing example

User: *"Make a cinematic shot of ancient Kolkata during the monsoon."* → the router decides
`TASK = cinematic T2V` and expands it into SCENE / SUBJECT / ENVIRONMENT / MOTION / CAMERA /
LIGHT / LENS / TEMPORAL / STYLE / NEGATIVE blocks before calling the video model.

## 2. Negative-prompt engine

Maintain **task-specific** negative prompts.

**Video**

> flicker, temporal instability, identity drift, face deformation, extra limbs, warped hands,
> duplicate subjects, unstable background, geometry deformation, texture swimming, camera
> jitter, lighting flicker.

**Image**

> anatomical errors, duplicate objects, distorted text, incorrect logos, bad perspective,
> unwanted objects.

**Audio**

> clipping, distortion, unwanted voice, digital artifacts, unnatural rhythm.

## 3. Rules

- **Model-aware:** tailor the prompt and negative prompt to the specific model's strengths
  and known failure modes — never reuse one prompt across modalities.
- **Separate concerns:** subject motion, camera motion and environment motion are specified
  independently.
- **Reproducible:** every prompt (and negative prompt) is stored in the shot spec / manifest
  with its seed and model version (see `master-specification.md`, `persistence-protocol.md`).
- **Text is never prompted** — reconstruct text via OCR, never let a generator invent it.

## Sources

Master specification v1.0 (prompt and negative-prompt engines, 2026).
