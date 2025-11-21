# Truck Geometry

## Overview

`truck-geometry` provides the spline primitives that power Truck’s CAD kernel. It exposes knot vectors, B-spline / NURBS curves, tensor-product surfaces, and refinement utilities that feed `truck-topology` and `truck-modeling`.

- Supported toolchain: Rust 1.70+ (confirm MSRV in `Cargo.toml`).
- License: MIT / Apache-2.0 dual (see `LICENSE`).
- Bug reports: open a GitHub issue and prefix titles with `[geometry]`.

## Resources

- [GitHub repository](https://github.com/ricosjp/truck)
- [docs.rs API reference](https://docs.rs/truck-geometry)
- `README.md` / `developer_docs/`: cross-module guidelines and checklists

When filing PRs, keep the English and Japanese guides in sync.

## Setup & Usage

### Build / Test

```
cargo test -p truck-geometry
cargo doc -p truck-geometry --open
```

Workflow automation is available through `cargo make` targets declared in `Makefile.toml`.

### Dependent crates

- `truck-base`: tolerance and numeric utilities shared across the workspace.
- `truck-topology`: consumes parameter ranges and trimming data.

## NURBS & Knot Vectors

- Maintain knots as non-decreasing, typically normalized to [0, 1]; clamp endpoints for CAD interoperability.
- Multiplicity directly controls continuity; re-check `is_smooth()` or similar helpers after edits.
- Use `insert_knot`, `remove_knot`, and `clamp` helpers rather than mutating the raw vector.

## Curves

Curves implement the standard `ParametricCurve3D` traits and rely on De Boor evaluation.

Typical API usage:

```
let p = curve.evaluate(t);
let dp = curve.der(t);
let d2 = curve.der2(t);
```

- Keep parameters inside `[knots[p], knots[n-p])` or normalize them before evaluation.
- NURBS uses homogeneous coordinates for computation; expose affine results only.
- Return `EvalError` for invalid parameters or degenerate weights rather than panicking.

## Surfaces

- Construct via tensor products: independent `u` / `v` degrees, knot vectors, and control nets.
- Provide `uder`, `vder`, and `uvder` for partial derivatives; normals are `uder × vder`.
- Expose accurate UV ranges for `truck-topology` trim loops.

## Control Net & Refinement

- Keep edits local—after moving a control point, re-evaluate only the affected span for predictable modeling.
- Degree elevation, knot insertion, and reparameterization utilities live in the refinement module; reuse them.
- Tolerance-sensitive code should accept a `truck-base` tolerance value instead of hard-coded epsilons.

For PRs, add or update unit tests under `tests/` and run `cargo fmt` before submitting.
