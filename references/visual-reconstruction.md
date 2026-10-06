## 7. CORE PROJECT OBJECTIVE

> **How to use this module.** Load when reconstructing source frames. Before locking a model, refresh the pick via `references/model-discovery.md`.

This is NOT a conventional image-upscaling project.

Do NOT treat the source as:

LOW QUALITY IMAGE
    ↓
PIXEL UPSCALER
    ↓
LARGE LOW QUALITY IMAGE

The objective is:

SOURCE
    ↓
UNDERSTAND
    ↓
RECONSTRUCT HIGH-QUALITY VISUAL
    ↓
DECOMPOSE
    ↓
SEGMENT
    ↓
REFINE
    ↓
EXTRACT
    ↓
GENERATE MISSING ELEMENTS
    ↓
OPTIONALLY RECONSTRUCT AS 3D
    ↓
REBUILD ORIGINAL COMPOSITION
    ↓
QUALITY CONTROL
    ↓
EDITABLE ASSET LIBRARY

The primary visual-quality stage MUST be
HIGH-QUALITY IMAGE GENERATION / IMAGE RECONSTRUCTION.

Do NOT use a conventional super-resolution model as the primary
quality stage.

---

## 8. IMAGE GENERATION — PRIMARY VISUAL QUALITY STAGE

Use a TOP-TIER practical local/open-weight image-generation or
image-editing/reconstruction model.

Do NOT automatically select a weak/lightweight model simply because
it is fast.

At runtime:

1. inspect GPU
2. inspect VRAM
3. inspect compatible model implementations
4. determine which high-quality model can actually run
5. choose the strongest practical option
6. use reference/image-to-image/editing conditioning whenever possible

QUALITY > FIDELITY > EDITABILITY > SPEED

---

## 9. PREFERRED IMAGE-GENERATION MODEL FAMILY

Consider strong compatible models such as:

- FLUX.2
- Qwen-Image
- Qwen-Image-Edit
- FLUX Kontext
- HunyuanImage
- HiDream
- Z-Image
- FLUX.2 Klein

Do NOT assume all models can fit the T4.

Choose the strongest practical model supported by the current
environment.

For approximately 16 GB T4-class hardware, prefer a configuration/model
that can run reliably through:

- quantization
- FP16 where supported
- CPU offloading
- memory-efficient attention
- tiled processing
- sequential loading

Potential practical candidates include:

- Z-Image-Turbo
- FLUX.2 Klein 4B
- compatible reference/editing models

If a stronger compatible model can run reliably, use it instead.

Do NOT let one unavailable model stop the project.

---

## 10. IMAGE RECONSTRUCTION PRINCIPLE

DO NOT generate an unrelated replacement frame.

The image-generation model must use the source as reference.

Preserve as much as possible:

- composition
- geometry
- visual identity
- object identity
- layout
- perspective
- framing
- color
- proportions
- visual hierarchy
- lighting relationships
- typography placement
- relative scale

The objective is to RECONSTRUCT missing detail,
not redesign the scene.

---

## 11. REFERENCE / IMAGE-TO-IMAGE / EDITING

When supported, use:

- image-to-image
- reference image conditioning
- inpainting
- masked editing
- structural conditioning
- image guidance

Prefer constrained reconstruction over unrestricted text-to-image.

Whenever possible provide the model with:

- original frame
- source crop
- best available reference
- mask
- surrounding context

For individual assets, use the source frame and element crop as
reference rather than generating unrelated assets from text alone.

---

## 12. VISUAL RECONSTRUCTION OUTPUT

For every source frame create:

1. original copy/reference
2. high-quality reconstructed image
3. reconstruction metadata
4. side-by-side or comparison preview

Record:

original_width
original_height
reconstructed_width
reconstructed_height
model
model_version
generation_parameters
processing_time
reference_method
