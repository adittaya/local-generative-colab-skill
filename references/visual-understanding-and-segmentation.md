## 13. VISUAL UNDERSTANDING

> **How to use this module.** Load when decomposing a reconstructed frame into editable assets. Before locking a model, refresh the pick via `references/model-discovery.md`.

Analyze every reconstructed frame.

Identify visual components such as:

- background
- environment
- subjects
- people
- objects
- products
- secondary objects
- UI
- interface panels
- icons
- logos
- charts
- graphics
- typography
- decorative graphics
- glows
- particles
- smoke
- light trails
- reflections
- shadows
- holograms
- energy effects
- atmosphere
- foreground elements

Use a capable vision-language/image-understanding model when practical.

If a large VLM cannot run safely on the T4:

use the strongest smaller compatible option or an appropriate
offloaded analysis method.

The purpose of this stage is semantic understanding and decomposition,
not merely generic object detection.

---

## 14. MAXIMUM EDITABILITY PRINCIPLE

THIS IS ONE OF THE MOST IMPORTANT REQUIREMENTS.

DO NOT aggressively minimize the number of assets.

The target is:

MAXIMUM USEFUL EDITABILITY.

If an element can independently be:

- moved
- scaled
- rotated
- recolored
- hidden
- replaced
- animated
- regenerated
- composited
- edited

then preferably make it a separate asset.

For example:

BACKGROUND
PERSON
CLOTHING
OBJECT
UI PANEL
ICON
CHART
TEXT
GLOW
LIGHT TRAIL
PARTICLES
REFLECTION
SHADOW
FOREGROUND EFFECT

may all be independent assets.

---

## 15. DO NOT OVER-FRAGMENT

Maximum editability does NOT mean meaningless pixel fragmentation.

Do NOT create useless assets consisting only of:

- compression noise
- random pixels
- accidental artifacts
- meaningless texture noise
- arbitrary anti-aliasing fragments

BUT:

Small intentional visual components SHOULD be preserved.

Examples:

- small icons
- small particles
- decorative dots
- indicators
- small UI controls
- light streaks
- isolated glows
- tiny graphical elements

A practical rule:

"If this region could reasonably be independently edited, moved,
animated, regenerated, recolored, replaced, or composited, consider
it a separate asset."

---

## 16. SAM 2.1 LARGE

Use:

SAM 2.1 Large

as the primary pixel-level segmentation/masking system.

SAM is the SEGMENTATION ENGINE.

SAM is NOT the sole semantic decision-maker.

Use semantic analysis to determine what each mask represents.

Do not blindly accept every automatic SAM mask.

At the same time:

DO NOT aggressively discard valid useful masks solely to reduce the
asset count.

The goal is MAXIMUM USEFUL DECOMPOSITION.

---

## 17. SEGMENTATION WORKFLOW

For every candidate visual element:

1. identify region
2. create segmentation mask with SAM 2.1 Large
3. refine mask if necessary
4. preserve fine edges
5. create alpha
6. save mask
7. save transparent asset
8. create preview

Do NOT simply produce rectangular crops.

---

## 18. BIREfNET REFINEMENT

Use:

BiRefNet

for additional foreground/mask refinement when useful.

Especially for:

- hair
- thin structures
- soft edges
- difficult boundaries
- semi-transparent edges
- complex foregrounds
- difficult object separation

Preferred pipeline:

SEMANTIC IDENTIFICATION
    ↓
SAM MASK
    ↓
BIREFNET REFINEMENT
    ↓
ALPHA
    ↓
TRANSPARENT PNG

---

## 19. TRANSPARENT EXTRACTION

The desired extracted asset is:

ACTUAL ELEMENT
+
TRANSPARENT BACKGROUND

NOT:

RECTANGULAR CROP
+
ORIGINAL BACKGROUND

For each visual extraction create:

- transparent PNG
- mask PNG
- preview PNG
- metadata JSON
