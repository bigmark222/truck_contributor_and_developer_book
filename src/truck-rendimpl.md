# Truck Rendimpl

## Overview

Module Guide
The high-level renderer for Truck geometry. Builds on `truck-platform` to provide CAD-friendly shading, selection, and camera control out of the box.

Crate: truck-rendimpl Backend: WGPU via truck-platform Highlights: Tess mesh upload, forward shading, selection buffers
## 1. Renderer layers

Rendimpl sits above truck-platform:

- **truck-platform** – device/queue/context + `Rendered` trait
- **truck-rendimpl** – defines concrete rendered objects for CAD (meshes, wires, selection, axes)
- **Samples/viewers** – use `truck-rendimpl` to visualize modeling/tessellation output

## 2. Render pipelines

### 2.1 Forward shading

Implements a configurable forward renderer:

- Supports multiple lights, toon/PBR-ish shading toggles.
- Handles per-face/vertex colors and normals from tessellation.
- Exposes uniform buffers for camera, lighting, and selection state.

### 2.2 Wireframe overlay

Draws mesh edges with optional depth offset to visualize topology.

### 2.3 Selection buffer

Rendimpl renders an off-screen buffer encoding IDs per primitive. Mouse picking reads this buffer to determine which face/edge/vertex is under the cursor.

## 3. Mesh upload

### 3.1 From tessellation

Meshes from `truck-meshalgo` are converted into GPU buffers:

- Vertex positions, normals, UVs.
- Index buffers for triangles/quads.
- Instance data for repeated geometry (see below).

### 3.2 Instancing

Instances encode per-object transforms, colors, and selection flags. Rendimpl uses instancing to render large scenes efficiently.

## 4. Interactivity & camera

Out-of-the-box camera controls (orbit/pan/zoom) and input handling integrate with the selection buffer and WGSL overlay. Viewers can focus on geometry and let rendimpl manage interaction plumbing.

## 5. Integration with other crates

- **truck-modeling / truck-shapeops** – produce solids whose tessellated meshes are uploaded here.
- **truck-platform** – supplies device/context and `Rendered` trait implementations.
- **truck-meshalgo / truck-polymesh** – provide mesh data ready for upload.

## 6. Best practices

- **Reuse GPU resources** – avoid recreating buffers when meshes only change transforms.
- **Synchronize selection state** – keep CPU-side IDs in sync with the selection buffer.
- **Coordinate conventions** – maintain Truck’s right-handed Z-up view so modeling output matches expectations.
- **Hot shader reload** – reuse truck-platform’s shader reload utilities when iterating on WGSL.

## 7. Summary

`truck-rendimpl` gives Truck a CAD-ready renderer: it understands tessellated meshes, draws wireframes, handles selection, and integrates seamlessly with the rest of the engine.

## 8. Implementation checklist

- **Init** – set up truck-platform context before constructing rendimpl objects.
- **Mesh data** – ensure tessellation outputs normals/UVs so shading works.
- **Selection IDs** – assign unique IDs per entity when uploading to selection buffer.
- **Docs** – update README/guide whenever new pipelines or viewer features are added.
