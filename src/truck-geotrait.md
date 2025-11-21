# Truck Geotrait

## Overview

Interface Guide
A tour of the traits that describe curves, surfaces, and parameter search—every geometry implementation in Truck must satisfy this contract.

Crate: truck-geotrait Consumers: geometry, topology, modeling, CSG Highlights: ParametricCurve/Surface, parameter search, transformation helpers
## 1. Design goals

`truck-geotrait` keeps geometry and topology loosely coupled:

- Geometry crates (e.g. `truck-geometry`) implement these traits.
- Topology/modeling crates consume them without knowing the concrete implementations.
- Procedural macros (`truck-derivers`) fill in forwarding code for wrapper types.

The traits emphasize:

- Consistent parameter domains (bounded or infinite).
- Derivative access for differential geometry tooling.
- Search/division interfaces so algorithms can adaptively sample geometry.

## 2. Shared helpers

### 2.1 Invertible & Transformed

These traits apply to both curves and surfaces:

- `Invertible` – reversible orientation (used by edges/wires to flip direction).
- `Transformed<Matrix3/4>` – apply rotations/affine transforms to control nets and derived points.

### 2.2 ToSameGeometry

`ToSameGeometry` returns an equivalent geometry object (e.g., boxing a concrete curve). This powers caching layers and type erasure in modeling.

### 2.3 Parameter ranges

The alias `ParameterRange = (Bound<f64>, Bound<f64>)` plus helper `bound2opt` unify how ranges are exposed. Bounded traits panic if either bound is `Unbounded`, making errors obvious.

## 3. Curve traits

### 3.1 ParametricCurve basics

Implementations map `t` to a point/vector space:

- `subs(t)`
- `der`, `der2`, `der_n`, `ders`
- `parameter_range` and optional `period()`

Type aliases (`ParametricCurve2D`, `ParametricCurve3D`) constrain point/vector types.

### 3.2 Bounded & transforms

- `BoundedCurve` – exposes `range_tuple()`, `front`, `back`.
- `ParameterTransform` – affine remap of parameter domains, with helpers to normalize to [0, 1].
- `Cut` – splits a curve at `t`, returning the trailing segment.

### 3.3 Parameter division

`ParameterDivision1D` returns adaptive sampling tuples `(Vec<t>, Vec<Point>)`.

### 3.4 Parameter search

Traits such as `SearchParameterD1` and `SearchNearestParameterD1` project points onto curves using Newton iterations. Higher-level algorithms (e.g. topology trimming) depend on these interfaces to be implemented correctly.

## 4. Surface traits

### 4.1 ParametricSurface

Mirrors the curve trait but with two parameters:

- `subs(u, v)`
- Partial derivatives (`uder`, `vder`, `uuder`, `uvder`, `vvder`, `der_mn`, `ders`)
- Range metadata for both `u` and `v`
- `ParametricSurface3D` adds `normal` + derivatives of normals

### 4.2 Bounded & include curve

- `BoundedSurface` – ensures finite parameter domains.
- `IncludeCurve<C>` – reports whether a curve lies on the surface (used to validate trims).

### 4.3 UV division & search

`ParameterDivision2D` produces sampling grids; `SearchParameterD2` projects onto surfaces.

## 5. Search helpers

The `traits/search_parameter.rs` module offers Newton-based solvers for parameter search. These are used by both curves and surfaces, and are exposed via the `Search*` traits. When implementing custom geometry, ensure your `ders` results are accurate so these solvers converge.

## 6. Integrating with truck-geometry

Concrete spline implementations in `truck-geometry` satisfy these traits. If you build new primitives:

- Return precise parameter bounds (normalized if necessary).
- Implement derivative routines needed by search/division.
- Use `truck-derivers` macros if your type just wraps existing geometry.

## 7. Derive macros

Enabling the `derive` feature in `truck-geotrait` pulls in `truck-derivers`, letting you do:

```
#[derive(ParametricCurve, BoundedCurve, SearchParameterD1)]
pub enum Curve {
    Line(LineCurve),
    Nurbs(NurbsCurve),
}
```

Each derive forwards methods to the inner type. See `docs/truck-derivers.html` for the full list.

## 8. Best practices

- **Tolerance** – use `truck-base::tolerance` helpers when comparing parameters.
- **Normalization** – keep parameter domains normalized unless native values are required for interoperability.
- **Zero checks** – use the `Origin` trait (`so_small`) for near-zero tests.
- **Documentation** – update README/traits docs when adding traits or changing contracts.

## 9. Summary

`truck-geotrait` abstracts geometry operations so the rest of the kernel can be agnostic about concrete curve/surface types. Implementations should:

- Provide evaluation/derivative access.
- Expose accurate parameter domains.
- Support adaptive sampling and projection via the search/division traits.
- Handle transforms and orientation changes.

## 10. Implementation checklist

- **Bounds** – confirm `parameter_range` (and bounded variants) return finite values where required.
- **Derivatives** – validate derivative accuracy; search routines depend on them.
- **Transforms** – implement both `Transformed<Matrix3>` and `Transformed<Matrix4>` if the geometry will live in 3D.
- **Derives** – use `truck-derivers` to avoid boilerplate on wrapper enums/structs.
- **Docs/tests** – add examples/tests for new trait impls and update this guide if APIs change.
