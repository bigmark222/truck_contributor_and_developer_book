# Truck Derivers

## Overview

Macro Guide
Procedural macros that implement the repetitive, boilerplate-heavy traits from `truck-geotrait`, keeping curve and surface types small and easy to maintain.

Crate: truck-derivers Consumers: truck-geotrait (feature = "derive") Targets: Parametric curves & surfaces, search helpers, STEP IO
These macros derive entire trait impls for wrapper enums or tuple structs. They are primarily invoked by `truck-geotrait` when you enable the `derive` feature, but you can reference them directly if your crate defines custom curve/surface types that still follow Truck’s trait contracts.

## 1. Getting started

Most users only need to flip on the derive feature in `truck-geotrait`:

```
[dependencies]
truck-geotrait = { version = "0.4", features = ["derive"] }
```

Doing so exposes the `#[derive(...)]` macros listed below.

### Wrapper pattern

Derives expect simple data shapes:

- Tuple structs wrapping a single field (e.g. `struct NurbsCurve(pub MyImpl);`)
- Enums whose variants each hold exactly one field (common for shape unions)

All variants must contain types that already implement the underlying trait. The macro simply forwards the trait methods to the inner type and re-wraps the result when necessary.

## 2. Curve trait derives

Derived curves typically wrap low-level implementations from `truck-geometry`.

### 2.1 Bounded & parametric

- `#[derive(ParametricCurve)]` – forwards evaluation (`subs`, `der`, etc.) and parameter metadata.
- `#[derive(BoundedCurve)]` – adds `range_tuple`, `front`, `back` while panicking if any variant exposes an unbounded range.
- `#[derive(ParametricCurve3D)]` is implied once the point/vector types are `Point3/Vector3`.

Use these derives whenever you are building high-level curve types (e.g. enums of line/arc/spline) that simply dispatch to concrete primitives.

### 2.2 Parameter division

`#[derive(ParameterDivision1D)]` forwards adaptive sampling logic. Each variant’s inner type must already implement `ParameterDivision1D`. The derive simply returns the inner tuple of parameters and evaluated points.

### 2.3 Parameter search

Truck uses dedicated traits for nearest-parameter queries:

- `SearchParameterD1`, `SearchNearestParameterD1` for 1D curves.
- The `D2` variants operate on 2D parameter spaces (useful for trimming curves on surfaces).

Derives (`#[derive(SearchParameterD1)]`, etc.) automatically dispatch to the inner solver, preserving diagnostics from `truck-geotrait::algo`.

### 2.4 Utilities

- `#[derive(Cut)]` – splits a bounded curve at a parameter and returns the trailing segment.
- `#[derive(Invertible)]` – flips orientation for each variant/tuple struct, using `Invertible::invert`.

These utilities keep topological algorithms simple: edges can reverse orientation or be trimmed without re-implementing the math per variant.

## 3. Surface trait derives

Surface derives mirror their curve counterparts but operate on two parameters.

### 3.1 Parametric & bounded

- `#[derive(ParametricSurface)]` – forwards all partial derivatives and range metadata.
- `#[derive(ParametricSurface3D)]` – adds normal computations and derivatives of normals.
- `#[derive(BoundedSurface)]` – exposes finite `u` and `v` ranges.

### 3.2 UV division

`#[derive(ParameterDivision2D)]` forwards two-dimensional sampling (used for tessellation and adaptive evaluation).

### 3.3 UV search

The `SearchParameterD2` and `SearchNearestParameterD2` derives funnel Newton-based projection routines from `truck-geotrait::algo` to all variants.

## 4. Transform & geometry derives

### 4.1 Matrix derives

- `#[derive(TransformedM3)]` – implements `Transformed<Matrix3>` by forwarding `transform_by` to the inner type.
- `#[derive(TransformedM4)]` – same but for full affine transforms (`Matrix4`).

These macros are essential when your wrappers need to respond to assembly-level transforms without bespoke code for each variant.

### 4.2 SelfSameGeometry

`#[derive(SelfSameGeometry)]` plugs into the `ToSameGeometry` trait, returning a boxed clone of the inner geometry. Use this when you want to erase the concrete type but still provide a canonical representation (e.g., caching surfaces by their base geometry).

## 5. STEP / export derives

Truck’s STEP writer relies on a small set of traits for formatting and entity length calculations.

### 5.1 StepCurve / StepSurface

- `#[derive(StepCurve)]` – forwards STEP serialization for curves.
- `#[derive(StepSurface)]` – same for surfaces.

Each variant’s inner type is expected to already implement the relevant STEP trait.

### 5.2 Display helpers

- `#[derive(StepLength)]` – indicates the number of STEP records generated.
- `#[derive(DisplayByStep)]` – builds user-friendly `Display` impls using STEP syntax.

## 6. Debugging derived impls

When a derive panics, the error usually points to one of these issues:

- **Variant shape** – ensure every variant holds exactly one field, with or without a named identifier.
- **Missing trait bounds** – add `where` clauses so the inner type implements the trait being derived.
- **Unbounded ranges** – the bounded derives panic if a variant reports `Bound::Unbounded`. Double-check the wrapped type.

Enable `RUSTFLAGS="--cfg proc_macro_span"` to surface better span information from `proc_macro_error` if necessary.

## 7. Summary

`truck-derivers` eliminates repetitive forwarding code, letting you:

- Wrap geometry primitives in enums/structs without hand-writing trait impls.
- Keep parameter search/division logic consistent across all curve/surface variants.
- Integrate transformation and STEP serialization behavior with a single attribute.

Whenever you add a new wrapper type or union of primitives, use these derives to stay compliant with the broader Truck kernel contracts.

## 8. Implementation checklist

- **Features** – enable `truck-geotrait` ’s `derive` feature (or depend on `truck-derivers` directly).
- **Shape** – verify each struct/enum variant is a single-field wrapper.
- **Bounds** – add `where` clauses so inner types implement the traits you derive.
- **Testing** – write regression tests for representative variants; ensure derived impls route to the correct inner behavior.
- **Docs** – update crate READMEs / guides whenever you add or remove derive macros.
