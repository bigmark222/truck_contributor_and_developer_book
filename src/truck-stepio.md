# Truck Stepio

## Overview

Module Guide
Bridges Truck solids with STEP files: parses ISO 10303 entities into Truck geometry/topology and exports back to STEP for interoperability with external CAD systems.

Crate: truck-stepio Formats: ISO 10303-21 (STEP Part 21) Tools: shape-to-step, step-to-mesh
## 1. Architecture

Stepio has three layers:

- **Parser** – reads Part 21 syntax into an intermediate entity graph.
- **Mapper** – converts STEP entities to Truck geometry/topology (and vice versa).
- **CLI** – sample binaries to convert shapes and inspect output.

## 2. Import pipeline

### 2.1 Parsing & dictionary

Part 21 text is parsed into a dictionary keyed by entity ID (e.g., `#42 = EDGE_CURVE...`). Relationships are resolved using references before geometry creation begins.

### 2.2 Geometry mapping

Each STEP entity maps to Truck types:

- `EDGE_CURVE`, `B_SPLINE_CURVE_WITH_KNOTS` → `ParametricCurve` / `truck-geometry` splines.
- `B_SPLINE_SURFACE_WITH_KNOTS`, `ADVANCED_FACE` → surfaces + trimming loops.
- Topological entities like `FACE_BOUND`, `SHELL_BASED_SURFACE_MODEL` → `truck-topology` faces/shells.

### 2.3 Healing hooks

Shape healing (snap vertices, align trim tolerances) will often run after import to handle noisy data from other CAD systems.

## 3. Export pipeline

### 3.1 Entity encoding

When writing STEP, truck-stepio walks shells/faces and emits the corresponding STEP entities in Part 21 syntax. It relies on the STEP traits implemented across geometry types (see below).

### 3.2 Record counts

Derived traits like `StepLength` report how many entities a curve/surface produces, allowing the writer to pre-allocate IDs or validate file size.

## 4. Step traits

Geometry types implement traits from `truck-stepio` (often derived by `truck-derivers`):

- `StepCurve`, `StepSurface` – format entity strings.
- `DisplayByStep` – render human-readable diagnostics.
- `StepLength` – entity counts.

These traits ensure STEP export stays consistent across all geometry types.

## 5. CLI tools

- `shape-to-step` – convert Truck JSON shapes to STEP.
- `step-to-mesh` – parse STEP into shapes and immediately tessellate.

Use these binaries as references when building custom import/export pipelines.

## 6. Best practices

- **Preserve tolerance** – carry STEP-specified tolerances into Truck to avoid snapping artifacts.
- **Log unknown entities** – report STEP entity types that lack mappings so they can be implemented later.
- **Validate loops** – after import, ensure trim loops are oriented correctly and cover the intended region.
- **Reuse traits** – when adding new curve/surface types, derive the STEP traits to keep serialization consistent.

## 7. Summary

`truck-stepio` handles STEP interoperability for Truck: parsing entity graphs, mapping to Truck geometry/topology, and exporting back to Part 21. It powers the CLI tools and underpins future CAD exchange features.

## 8. Implementation checklist

- Test new entity mappings with both import and export.
- Keep tolerance handling consistent with `truck-base`.
- Document new STEP traits or CLI flags in README and this guide.
- Run `shape-to-step` / `step-to-mesh` as smoke tests when making changes.
