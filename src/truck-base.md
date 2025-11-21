# Truck Base: Geometry Foundations

## Overview

**Module guide.** truck-base documents the conventions that keep every Truck crate aligned.

- **Crate:** truck-base
- **Scalar:** f64
- **Coordinates:** right-handed, Z-up
- **Units:** meters (recommended)

`truck-base` defines the fundamental language of geometry for the Truck CAD kernel. It standardizes:

- Coordinate systems and spatial conventions
- Point, vector and matrix types (via `cgmath64`)
- Numeric tolerance strategy
- Transformation composition and application

All higher-level crates (curves, surfaces, topology, booleans, meshing, constraints) must comply with these rules. Understanding this module is a prerequisite for implementing or extending geometry algorithms within the Truck ecosystem.

## 1. Coordinate systems

Truck uses a right-handed, Z-up Cartesian coordinate system, mirroring `cgmath` and common CAD kernels such as OpenCascade and Parasolid.

### 1.1 Right-handed system

The system is oriented as follows:

- **+Z** points “up”.
- Rotation from **+X** toward **+Y** follows the curl of the right hand.
- Positive angular motion is counter-clockwise when looking down +Z.

### 1.2 World vs local space

Truck distinguishes between two types of frames:

- **World space** – a single, global frame for the entire model or scene.
- **Local space** – frames attached to entities such as curves, surfaces or components.

Examples of local frames include:

- A curve’s parameter space, where `t` is mapped to a 3D point.
- A surface’s `(u, v)` coordinates describing a patch.
- An assembly sub-component’s local origin and orientation.

Local coordinates are always interpreted relative to their parent frame and must be converted to world coordinates via Truck’s transform system (see [Section 4](#transforms)).

### 1.3 Homogeneous coordinates

Internally, Truck uses homogeneous coordinates `[x, y, z, w]` to implement affine transforms. However:

- Public APIs expose only affine points and vectors:

```
Point2, Point3
Vector2, Vector3
```

The `w` component is an implementation detail. User code should not depend on or manipulate homogeneous representations directly.

## 2. Tolerance strategy

Direct equality comparisons on floating-point values are not reliable in geometric contexts (e.g. incidence tests, closure checks, intersection predicates). Truck centralizes numerical tolerances in the `tolerance` module to ensure consistent behavior.

### 2.1 Global tolerances

Truck defines the following constants:

```
TOLERANCE  = 1e-6
TOLERANCE2 = (1e-6)^2
```

- `TOLERANCE` is used for distances and scalar differences.
- `TOLERANCE2` is used for squared distances.

When units are interpreted as meters, `TOLERANCE` is approximately one micrometer.

### 2.2 Traits and helpers

All equality and incidence checks in higher-level crates should be implemented via the provided helpers:

- `Tolerance` trait
- Methods such as `Tolerance::near` and `Tolerance::near2`
- Assertion macros: `assert_near!`, `assert_near2!`

These utilities ensure that geometric predicates behave consistently, regardless of where they appear in the kernel.

### 2.3 Tightening vs loosening tolerances

**Guideline:** Normalize inputs; don’t compensate with looser tolerances.
Truck follows two key rules:

- **Do not loosen** tolerances to accommodate low-quality input. Instead, pre-process and clean the geometry before it enters the kernel.
- **Only tighten** tolerances when a specific algorithm demands it, and document the rationale (e.g. highly sensitive intersection routines).

This prevents subtle inconsistencies between different modules and makes robustness issues easier to diagnose.

### 2.4 Zero/origin checks

The `Origin` trait extends `Tolerance` with helpers such as `so_small` and `so_small2`, which report whether a scalar, vector, or point is effectively zero within the global tolerance. Favor these helpers when testing for “origin-ness” (e.g. near-singular normals, degenerate vectors) instead of hand-rolled checks.

## 3. Points, vectors & normals

Truck re-exports `cgmath` primitives specialized to `f64` through the `cgmath64` module:

```
Point2, Point3
Vector2, Vector3
Matrix3, Matrix4
Quaternion
```

### 3.1 Affine semantics

Truck relies on CGMath’s type system to distinguish between positions and directions:

- **Point** – a location in space.
- **Vector** – a direction or displacement.

Valid combinations include:

- `Point3 + Vector3 → Point3`
- `Point3 - Point3 → Vector3` (displacement)
- `Vector3 + Vector3 → Vector3`

Operations such as adding two points directly are intentionally disallowed. This eliminates a wide class of geometric mistakes.

### 3.2 Normals

Surface normals are represented as normalized `Vector3` values (unit vectors). Truck provides extension traits (e.g. `cgmath_extend_traits`) to support:

- Normalization and validation
- Conversions to/from homogeneous coordinates
- Derivative-based helpers for differential geometry (via `ders`)

Normals should always be kept unit-length and never treated as points.

### 3.3 Indexing

By convention:

- `v[0] = x`
- `v[1] = y`
- `v[2] = z`

Any `w` component exists only in internal homogeneous representations and is not part of the public API.

## 4. Transformation system

Transforms in Truck are represented using CGMath matrices and quaternions, all available through the `cgmath64` re-exports:

- `Matrix3` – rotation and uniform scaling
- `Matrix4` – full affine transforms (rotation, translation, scale)
- `Quaternion` – for representing rotations compactly

### 4.1 Construction

Typical constructors include:

```
use truck_base::cgmath64::{Matrix4, Vector3, Point3};

let translation = Matrix4::from_translation(Vector3::new(1.0, 2.0, 3.0));
let scale       = Matrix4::from_nonuniform_scale(2.0, 2.0, 2.0);

let transform   = translation * scale;

let p_local = Point3::new(0.0, 0.0, 0.0);
let p_world = transform.transform_point(p_local);
```

These helpers should be preferred over manually assembling matrices, as they are easier to read and less error-prone.

### 4.2 Transform traits

Transforms are applied using CGMath’s traits such as `Transform` and `EuclideanSpace`:

```
let p_world = transform.transform_point(p_local);
let v_world = transform.transform_vector(v_local);
```

This ensures that:

- Points receive rotation and translation.
- Vectors receive rotation but not translation.

### 4.3 Composition order

Truck composes transforms in parent-to-child order. If `world_from_local` transforms local coordinates into world space, and `local_from_shape` transforms shape space into local space, then:

```
let world_from_shape = world_from_local * local_from_shape;
```

Conceptually, the rightmost transform is applied first. This ordering is important for assemblies, nested components, and hierarchical modeling.

### 4.4 Serialization

Truck and CGMath use **column-major** storage for matrices. Any serialized form (JSON, binary, database records, etc.) should preserve column-major ordering to ensure correct round-trips between in-memory and persisted representations.

## 5. Summary

`truck-base` defines the core rules that keep the Truck geometry kernel stable, predictable and mathematically coherent:

- Right-handed, Z-up coordinate system aligned with `cgmath`
- Consistent tolerance strategy via the `tolerance` module
- Strict separation between points, vectors and normals
- Well-defined transform construction, application and composition

All higher-level crates and algorithms must respect these conventions. Doing so ensures that curve/surface evaluation, boolean operations, meshing and future extensions – including experimental “sci-fi” features – operate on a shared, well-defined geometric foundation.

## 6. Implementation checklist

Before sending geometry/core-kernel PRs, sanity-check the following:

- **Frames** – confirm world/local transforms follow the parent → child multiplication order.
- **Tolerances** – use `Tolerance` / `Origin` helpers; avoid ad-hoc epsilons.
- **Types** – keep points, vectors, and normals distinct (no casting via raw tuples).
- **Matrices** – serialize/store in column-major order to match cgmath.
- **Docs** – update crate README/guide sections (like this file) when conventions change.

Following this checklist keeps the documentation accurate for newcomers while giving experienced engineers a reliable reference.
