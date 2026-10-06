# 3D Asset Generation — Hunyuan3D 2.1

> **How to use this module.** Load when the pipeline reaches the 3D branch. Before locking a model, refresh the pick via `references/model-discovery.md`.

The project has a dedicated **3D asset generation branch** for elements that are genuinely
volumetric, or that would significantly benefit from an editable 3D representation.

## Model

**Tencent Hunyuan3D 2.1** — a fully open-source 3D asset creation system (weights **and**
training code released). It is built from two foundation models:

- **Hunyuan3D-DiT** (`Hunyuan3D-Shape-v2-1`, 3.3B) — image → shape. A flow-based diffusion
  architecture paired with a high-fidelity mesh autoencoder (ShapeVAE) produces the geometry.
- **Hunyuan3D-Paint** (`Hunyuan3D-Paint-v2-1`, 2B) — mesh-conditioned multi-view diffusion
  that generates **PBR materials**: albedo, metallic and roughness maps, spatially aligned
  and view-consistent (3D-aware RoPE, illumination-invariant albedo for light-free maps).

Output is a **GLB mesh**, optionally with PBR materials.

## When to use it

Use Hunyuan3D 2.1 for elements that are **genuinely volumetric** or that gain real value
from being editable in 3D — props, products, characters, mechanical parts, set pieces.

Do **not** send flat elements here: UI panels, icons, text, logos, glows and particles are
better served by the 2D extraction branch (SAM 2.1 + BiRefNet → transparent PNG).

## Hardware & VRAM

| Stage | VRAM |
|---|---|
| Shape generation | ~10 GB |
| Texture (PBR) generation | ~21 GB |
| Shape + texture together | ~29 GB |

- On a **16 GB T4**: shape generation fits on its own; texture generation needs
  `--low_vram_mode` and CPU offloading. Run the two stages **sequentially** rather than
  holding both models resident — this matches the skill's sequential-loading rule.
- Always launch with the low-VRAM flag on constrained GPUs:
  `python3 gradio_app.py --model_path tencent/Hunyuan3D-2.1 --subfolder hunyuan3d-dit-v2-1 --texgen_model_path tencent/Hunyuan3D-2.1 --low_vram_mode`

## Pipeline

1. Select 3D candidates from the visual-understanding / 2D-vs-3D decision stage.
2. Keep the **source frame and element crop** as the image reference.
3. **Shape** — `Hunyuan3DDiTFlowMatchingPipeline` → untextured mesh.
4. **Texture** — `Hunyuan3DPaintPipeline` (multi-view, e.g. `max_num_view=6`,
   `resolution=512`) → PBR-textured mesh.
5. Export **GLB** (and the PBR variant where needed).
6. Render a turntable **preview** and write **metadata JSON**.

## Quality control

- Geometry: silhouette match to the source crop, no holes or spikes.
- Texture: PBR maps present and aligned; no seams or ghosting.
- Scale/orientation: consistent with the reconstructed scene.
- Keep the untextured mesh alongside the textured one — it is useful for relighting.

## Sources

Tencent Hunyuan3D-2.1 repository, model card and paper ("From Images to High-Fidelity 3D
Assets with Production-Ready PBR Material", arXiv 2506.15442).
