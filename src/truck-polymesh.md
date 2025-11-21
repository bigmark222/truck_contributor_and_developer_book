# Truck Polymesh

## Overview

Module Guide
Defines the half-edge mesh representation used throughout Truck for tessellation, rendering, and mesh-processing algorithms.

Crate: truck-polymesh Kernel role: Mesh data model Highlights: Half-edge structure, tess integration, mesh algorithms
## 1. Half-edge basics

The half-edge structure splits every edge into two directed half-edges. Each half-edge knows:

- Its origin vertex.
- The adjacent face.
- The opposite half-edge (twin).
- The next/prev half-edge around its face loop.

This design makes neighborhood queries (one-ring, star, opposite faces) constant time, and supports operations such as edge flips, collapses, and propagation of attributes like UVs.

## 2. Data structures

### 2.1 Vertices

Vertices store positions, optional normals, and a pointer to one outgoing half-edge. You can traverse all incident faces/edges by following `half_edge → twin → next`.

### 2.2 Half-edges

Each half-edge holds indices for `origin`, `twin`, `next`, `prev`, and its incident face. Attribute storage (boundary flags, UV direction) lives here too, making it easy to propagate data during tessellation or boolean operations.

### 2.3 Faces & loops

Faces reference one of their bounding half-edges. Since loops can contain holes, inner loops are represented by chains of half-edges whose face pointers all reference the outer face. Traversal follows the standard half-edge “next” pointers.

## 3. Feeding tessellation

Results from `truck-meshalgo` or analytic meshing pipelines emit vertices/faces in the `truck-polymesh` format:

- Vertices store world-space positions and optional UVs or normals.
- Edges capture adjacency so shading and boolean operations remain watertight.
- Supports both triangle and quad faces; higher valence polygons can be stored or triangulated later.

## 4. Algorithms & utilities

While most algorithms live in `truck-meshalgo`, this crate provides utility functions:

- Traversal helpers (neighbors, loops, boundary detection).
- Attribute propagation (copy normals/UVs along shared edges).
- Conversion helpers (OBJ import/export for debug, used by the sample tools).

## 5. Integration points

- **truck-modeling** → **truck-meshalgo** → **truck-polymesh**: shapes are tessellated into meshes.
- **truck-platform/truck-rendimpl** consume `truck-polymesh` for rendering.
- Mesh-processing examples (loop subdivision, smoothing) manipulate this data structure directly.

## 6. Best practices

- **Keep half-edge invariants** – whenever you edit a face/edge, update twin/next/prev pointers.
- **Store UVs** – tessellation should capture UV coordinates so trimming and texturing remain possible.
- **Use consistent winding** – follow Truck’s orientation rules (usually CCW for outward normals).
- **Validate** – run manifold/boundary checks after editing meshes to catch topology errors early.

## 7. Summary

`truck-polymesh` provides a robust half-edge mesh that underpins tessellation, rendering, and experimental mesh-processing tools. Understanding its structure makes it easier to integrate new algorithms or diagnostics into the Truck pipeline.

## 8. Implementation checklist

- Use half-edge helpers when adding/removing faces to avoid pointer mistakes.
- Ensure tessellated data includes normals and UVs if downstream consumers need them.
- When exporting, preserve vertex order and attributes (OBJ helpers are provided).
- Document new mesh utilities or attribute conventions in the README and this guide.
