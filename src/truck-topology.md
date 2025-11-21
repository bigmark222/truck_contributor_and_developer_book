# Truck Topology

## Overview

Module Guide
Defines boundary representation (B-rep) structures—vertices, edges, wires, faces, shells, and solids—and enforces the orientation rules used across the Truck kernel.

Crate: truck-topology Focus: B-rep data model Consumers: modeling, booleans, tessellation
## 1. Topology stack

Truck’s topology sits between geometry and mesh layers:

- **Geometry** – curves/surfaces from `truck-geotrait` / `truck-geometry`.
- **Topology** – organizes geometry into vertices, edges, wires, faces, shells, solids.
- **Modeling/Booleans** – operate on topology to create/manipulate solids.
- **Tessellation** – consumes topology’s trimmed surfaces to produce meshes.

## 2. Data structures

### 2.1 Vertex

Stores a 3D point plus references to incident edges. Vertices may also cache parameter-space coordinates for associated edges/surfaces.

### 2.2 Edge

Edges reference a parametric curve, two vertices, and an orientation. They may also store UV curves for trimming surfaces. Each edge has both 3D and parameter-space representations.

### 2.3 Wire

A closed sequence of oriented edges forming a loop. Wires can serve as outer loops (defining the boundary of a face) or inner loops (holes). Consistent orientation (CCW for outer loops) is required.

### 2.4 Face

Faces reference a surface plus an outer wire and zero or more inner wires. They encapsulate trimmed surface patches.

### 2.5 Shell & solid

Shells are closed, oriented sets of faces. Solids consist of one outer shell and optional inner shells (voids). Booleans operate at the shell/solid level.

## 3. Orientation & trims

Orientation rules keep the data model consistent:

- Edges have a “forward” direction based on their parametric curve.
- Wires are oriented so outer loops are CCW in UV space (relative to surface normal), inner loops CW.
- Face orientation determines shell normals; consistent sewing ensures outward normals on shells.

## 4. Integration with geometry & modeling

Topology relies on geometry for actual shape evaluation:

- Edges reference `ParametricCurve` implementations.
- Faces reference `ParametricSurface` plus wires that live in their parameter space.
- Modeling operators (extrude, revolve) generate topology in this format, ensuring compatibility with booleans and tessellation.

## 5. Best practices

- Ensure wires are closed and consistently oriented before constructing faces.
- Maintain edge metadata linking to source curves for downstream operations (e.g., booleans, fillets).
- Use the provided traversal utilities to navigate loops/shells instead of manual pointer chasing.
- Validate shells for watertightness/orientation before running booleans or tessellation.

## 6. Summary

`truck-topology` provides the B-rep backbone for Truck. It organizes geometry into a structured hierarchy that modeling, booleans, and tessellation can all consume.

## 7. Implementation checklist

- Verify edge orientations and wire closure before building faces.
- Attach both 3D and UV representations to edges/faces when trimming surfaces.
- When adding new topology features, update README/docs to explain orientation and traversal rules.
- Use integration tests (e.g., modeling scripts) to confirm new topology constructs behave through the entire pipeline.
