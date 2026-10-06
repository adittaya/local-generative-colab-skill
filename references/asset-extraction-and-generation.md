## 20. COMPLEX VISUAL ASSET CLASSIFICATION

> **How to use this module.** Load when deciding extract-vs-generate for each element. Before locking a model, refresh the pick via `references/model-discovery.md`.

Classify an element as:

COMPLEX_GENERATIVE_ASSET

when its appearance significantly depends on:

- cinematic lighting
- realistic materials
- photorealistic detail
- detailed 3D
- volumetric lighting
- glow
- bloom
- reflections
- refractions
- glass
- metal
- smoke
- fire
- water
- particles
- bokeh
- atmospheric haze
- holographic effects
- energy effects
- light trails
- intricate textures
- realistic shadows
- complex illumination
- realistic transparency
- cinematic depth
- advanced futuristic effects

---

## 21. NO PROCEDURAL RECREATION OF COMPLEX VISUALS

Do NOT reproduce complex visual assets using:

HTML
CSS
SVG
Canvas
basic Python drawing
simple gradients
basic geometric primitives
simple procedural graphics

These methods are allowed ONLY for genuinely simple elements such as:

- rectangles
- circles
- straight lines
- simple borders
- flat panels
- basic arrows
- simple charts
- basic icons

Do not simplify a complex rendered element merely because its
silhouette happens to contain geometric shapes.

---

## 22. GENERATION OF NON-EXTRACTABLE ASSETS

If an element cannot be accurately extracted:

DO NOT discard it.

Set:

generation_required = true

Create a dedicated generation specification.

Use the strongest practical image-generation/editing model.

The generated replacement must match the source element as closely
as practical.

The prompt should contain:

- subject
- shape
- geometry
- proportions
- material
- texture
- color
- lighting
- perspective
- camera angle
- depth
- transparency
- reflections
- refractions
- glow
- shadows
- atmosphere
- orientation
- visual style
- approximate scale

Also create:

negative_prompt

Do not include irrelevant scene information.

---

## 23. EXTRACTION VS GENERATION DECISION

For every element:

IF accurate extraction is possible:
    EXTRACT IT.

IF extraction is possible but edges are difficult:
    EXTRACT + REFINE.

IF the element is partly extractable:
    EXTRACT RELIABLE PART
    +
    GENERATE MISSING PART.

IF background and element are strongly intertwined:
    USE MASKED / REFERENCE RECONSTRUCTION.

IF extraction would badly damage the visual:
    GENERATE A REPLACEMENT USING THE SOURCE AS REFERENCE.

Do NOT regenerate an asset unnecessarily.

---

## 24. GENERATED TRANSPARENT ASSETS

If the image-generation model supports native transparency:

use native transparency.

Otherwise:

1. generate the asset
2. create/refine foreground mask
3. use SAM and/or BiRefNet
4. remove background
5. produce alpha
6. export transparent PNG
7. preserve separate mask

---

## 25. TEXT — HIGH PRIORITY

TEXT MUST NOT BE HALLUCINATED.

Image-generation models must NOT be trusted for exact source text.

For every text element:

1. detect it
2. OCR it
3. preserve exact wording
4. store typography information
5. extract/reconstruct separately when appropriate

Record:

exact_text
ocr_confidence
bbox_pixels
bbox_normalized
font_size_estimate
font_weight
font_style
alignment
color
opacity
rotation
line_spacing
letter_spacing
text_effects

If uncertain:

ocr_uncertain = true

Never invent missing characters.

---

## 26. LOGOS

Treat logos separately.

Attempt:

- exact extraction
- transparency
- high-quality mask
- OCR/text identification when applicable

If exact reconstruction cannot be achieved:

mark the logo for generation/reconstruction.

Never hallucinate or invent a logo.
