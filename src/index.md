# Truck Developer Docs

## Overview

Truck is a modular CAD kernel. Each crate has its own quick-reference page in this directory; use them to align on coordinate conventions, APIs, and contributor expectations before opening PRs.

- **Updated:** Continuous alongside the repo.
- **Audience:** Kernel and tooling engineers.
- **Stack:** Rust, cgmath64, WGPU, STEP.

## Module guides

- **[truck-base](truck-base.md)** — coordinate system, tolerance strategy, and transform conventions.
- **[truck-geotrait](truck-geotrait.md)** — parametric curve/surface traits and search helpers.
- **[truck-geometry](truck-geometry.md)** — knot vectors, B-spline/NURBS curves, surfaces, and refinement.
- **[truck-topology](truck-topology.md)** — B-rep data structures and orientation rules.
- **[truck-modeling](truck-modeling.md)** — sweeps, shells, and solid builders.
- **[truck-shapeops](truck-shapeops.md)** — boolean operations, healing, and diagnostics.
- **[truck-meshalgo](truck-meshalgo.md)** — tessellation, quadrangulation, and smoothing.
- **[truck-polymesh](truck-polymesh.md)** — the shared half-edge mesh structure.
- **[truck-platform](truck-platform.md)** — WGPU runtime, scene graph, and WGSL sandbox.
- **[truck-rendimpl](truck-rendimpl.md)** — CAD-specific renderer built on truck-platform.
- **[truck-derivers](truck-derivers.md)** — procedural macros for geometry traits and STEP derives.
- **[truck-stepio](truck-stepio.md)** — STEP import/export mappings.

## Resources

- [GitHub repository](https://github.com/ricosjp/truck)
- [Japanese docs](japanese/index.md) (mirrors this structure)
- Update both languages when APIs change; keep README pointers in sync.

## Workspace dependency graph

Generated from `cargo depgraph` so you can quickly orient yourself before diving into a crate.

![Truck workspace dependency graph](assets/deps.svg)
