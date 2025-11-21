# Truck Meshalgo

## Overview

Module Guide
Algorithms for tessellating trimmed NURBS, refining meshes, generating normals, and experimenting with mesh decomposition before exporting to downstream consumers such as rendering and analysis tools.

Crate: truck-meshalgo Inputs: truck-modeling / truck-topology shapes Outputs: truck-polymesh meshes, OBJ samples
## 1. Architecture overview

Meshalgo sits between analytic CAD data and polygonal consumers:

- Receives analytic geometry via `truck-geometry` & `truck-modeling`.
- Converts faces (with trims) into `truck-polymesh` half-edge meshes.
- Runs refinement passes (subdivision, quadrangulation, smoothing) depending on the workflow.
- Exports to file formats (OBJ, JSON) or directly to visualization crates.

Each algorithm is exposed through the library API and mirrored by sample binaries demonstrating usage.

## 2. Tessellation pipeline

Tessellation is the main entry point for converting shapes to meshes.

### 2.1 Sampling strategy

- Adaptive sampling based on curvature, trimming density, and user-supplied tolerances.
- Parameter domains come from `ParametricSurface` implementations; normalized domains are re-scaled to actual UV ranges.
- Edge isolines guarantee that mesh boundaries align with face trims.

Key tuning knobs include chordal deviation, angular tolerance, and maximum edge length. These mirror CAD tessellators, making it easier to match external kernels.

### 2.2 Trimming & topology

Trimming curves are evaluated in UV space and clipped via polygon boolean operations before being mapped back to 3D. The resulting tess is stitched into a watertight `truck-polymesh` and inherits edge orientation rules from `truck-topology`.

### 2.3 Export formats

Tessellated meshes can be exported as OBJ/JSON (see `tessellate-shape` example) or passed directly to rendering. Always preserve per-vertex normals / UVs if the downstream consumer expects them.

## 3. Subdivision & refinement

### 3.1 Loop subdivision

`octahedron-subdivision` demonstrates classic Loop subdivision: each triangle is split and vertex positions are averaged using Loop weights to create smooth approximations.

### 3.2 Quadrangulation

Examples like `requadrangulate-buddha` and `teapot` show how the crate converts triangle soups into quad meshes using dual graph techniques plus smoothing passes. This is useful for downstream NURBS fitting or quad-based render pipelines.

### 3.3 Mesh decomposition

Experiments such as `filleted-cube` and `splitting-sample` decompose meshes into planar or curved patches to aid feature recognition (future NURBS reverse-engineering). Although these binaries are exploratory, they demonstrate how to walk the mesh graph and group faces by curvature.

## 4. Smoothing & normal generation

Binaries like `smoothing-bunny` and `irregular-sphere` recompute vertex normals:

- Area-weighted or angle-weighted averaging for smooth shading.
- Optional projection back onto analytic surfaces to eliminate noise.

Use these routines when importing low-quality meshes or when tessellation produced per-face normals only.

## 5. Integration with other crates

- **truck-modeling / truck-shapeops** – provide analytic shapes to tessellate.
- **truck-polymesh** – receives final meshes using the half-edge structure expected by visualization/rendering.
- **truck-platform / truck-rendimpl** – render tessellated results (see OBJ outputs used by sample viewers).

Always keep consistency in tolerance usage: reuse `truck-base::tolerance` thresholds when snapping or welding vertices.

## 6. CLI samples

Sample binaries serve as living documentation. Highlights:

- `tessellate-shape` – `tessellate-shape input.json output.obj`
- `requadrangulate-buddha` – stress-tests quadrangulation + smoothing.
- `teapot` / `smoothing-bunny` – normal generation examples.
- `octahedron-subdivision` – minimal reproduction of Loop subdivision.

Refer to the README for the full list and input/output expectations.

## 7. Best practices

- **Watertightness** – weld vertices and check manifold conditions after tessellation.
- **Parameter tracking** – store UV coordinates per vertex when trimming; modeling/texture workflows depend on them.
- **Curvature-aware sampling** – tie tessellation density to `truck-geotrait` derivative magnitudes to avoid aliasing.
- **Performance** – use spatial subdivision (e.g., BVHs) when running operations on large meshes (see `requadrangulate-buddha`).

## 8. Summary

`truck-meshalgo` is the bridge from analytic geometry to polygonal representations:

- Robust tessellation with trimming support.
- Subdivision, quadrangulation, and decomposition utilities.
- Normal smoothing and experimentation tools for reverse engineering meshes.

## 9. Implementation checklist

- **Parameters** – expose knobs for chordal/angle tolerances; document defaults.
- **Normals** – recompute and normalize after every refinement step.
- **Metadata** – propagate UVs, materials, and edge tags through tessellation.
- **Validation** – assert manifoldness or report diagnostics when a mesh operation fails.
- **Docs** – update README/examples plus this guide when new algorithms land.
