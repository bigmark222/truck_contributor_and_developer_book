# Truck Shapeops

## Overview

Module Guide
Performs B-rep Boolean operations (AND/OR/XOR/Subtract) on shells and solids produced by `truck-modeling`, and contains healing utilities to prepare imported CAD geometry.

Crate: truck-shapeops Inputs: Solids from truck-modeling Highlights: Intersection solvers, classification, rebuild, healing tools
## 1. Boolean pipeline

The pipeline mirrors classic B-rep kernels:

1. **Intersections** – find all face/edge intersections between operands.
2. **Classification** – classify split faces/edges as inside/outside per Boolean op.
3. **Rebuild** – assemble new shells from the kept fragments.

## 2. Intersection stage

### 2.1 Face-face intersections

Surface-surface intersections yield curves that split faces and become new trimming loops.

- Curves are represented in both UV spaces; clipping occurs before remapping to 3D.
- Newton-style refinement ensures curves hit intersection tolerance (`truck-base::TOLERANCE`).

### 2.2 Edge-face intersections

Handles cases where an edge pierces the other solid without a clean face-face intersection; ensures loops remain consistent.

## 3. Classification

Each fragment (face region, edge segment) is classified via point-in-solid queries:

- Ray casting or signed distance proxies determine inside/outside.
- Operator rules (AND/OR/XOR/SUBTRACT) pick which fragments survive; normals flip as needed.

## 4. Rebuild & trimming

After classification:

- Face trims are clipped to the kept regions.
- Edges are sewn back into loops, respecting orientation.
- Shells are formed and validated for watertightness before returning the resulting solid(s).

## 5. Healing & diagnostics

Shapeops also includes utilities for importing external CAD data:

- Welding vertices within tolerance.
- Fixing small gap trims.
- Reporting inconsistent orientations or non-manifold conditions.

Run these tools before booleans when working with data from other kernels.

## 6. Best practices

- **Consistent tolerances** – align on `truck-base::tolerance` to avoid mismatched comparisons.
- **Trim orientation** – keep outer loops CCW (surface normal perspective) and inner loops CW.
- **Intersection filtering** – use bounding boxes / spatial partitioning to skip obvious non-intersections.
- **Diagnostics** – log when fragments fail classification to help debug kernel issues.

## 7. Summary

`truck-shapeops` performs the heavy lifting for CSG in Truck, splitting, classifying, and rebuilding shells to produce watertight Booleans while also offering healing utilities for imported data.

## 8. Implementation checklist

- Validate shells before entering the pipeline (orientation, manifoldness).
- Tune intersection tolerances per operator and document defaults.
- Ensure classification uses stable point-in-solid logic (avoid ray/geometry degeneracies).
- Update README/guide when new operations or healing tools are added.
