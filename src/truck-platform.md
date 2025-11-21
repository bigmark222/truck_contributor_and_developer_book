# Truck Platform

## Overview

Module Guide
Provides the WGPU runtime abstractions needed to render Truck’s CAD data—scene graph helpers, input handling, and the WGSL sandbox for rapid shader iteration.

Crate: truck-platform Backend: WGPU Highlights: Rendered trait, WGSL sandbox, scene graph utilities
## 1. Architecture

Platform wraps `wgpu` into higher-level constructs:

- Device/queue setup and swapchain management.
- Event loop integration for interactive viewers.
- Trait-based rendering (`Rendered`) so custom structs can upload buffers and draw.
- Utility components (camera controllers, input state, GUI overlays) shared by Truck viewers.

## 2. Rendered trait

The `Rendered` trait is the heart of custom rendering:

### 2.1 Lifecycle hooks

- `fn new(renderer: &RendererContext) -> Self` – create pipelines, buffers, textures.
- `fn update(&mut self, dt: f32)` – update uniforms, animations.
- `fn render(&self, pass: &mut wgpu::RenderPass)` – issue draw calls.

### 2.2 Resource management

Platform provides helpers for:

- Uploading vertex/index buffers from `truck-polymesh`.
- Creating bind groups for uniforms and textures.
- Sharing pipelines/shaders across multiple rendered objects.

## 3. WGSL sandbox

The `wgsl-sandbox` example demonstrates how to hot-reload WGSL shaders and render procedural content.

### 3.1 Shader contract

```
fn main_image(coord: vec2 , env: Environment) -> vec4
```

Shadertoy-style entry point: implement `main_image` to produce color output.

### 3.2 Environment inputs

```
struct Environment {
    resolution: vec2 ;
    mouse: vec4 ;
    time: f32;
}
```

Equivalent to Shadertoy’s `iResolution`, `iMouse`, and `iTime`.

### 3.3 Workflow

- Launch the binary with a WGSL file path, or drag-and-drop a shader onto the window.
- Hot reload recompiles the shader and restarts rendering immediately.
- Examples such as `newton-cuberoot.wgsl` serve as templates.

## 4. Scene graph & viewer

Platform offers building blocks for interactive CAD viewers.

### 4.1 Instances & buffers

- Batch instance data (transform, color) into structured buffers.
- Support for forward/deferred render paths (see `truck-rendimpl`).
- Utilities for uploading tessellated meshes or analytic primitives.

### 4.2 Materials & pipelines

Common material definitions (PBR-ish shading) live here so downstream crates can focus on geometry.

## 5. Integration with truck-rendimpl

`truck-rendimpl` builds on platform: it supplies CAD-specific rendered objects (e.g., edges, face shading, selection). Understanding platform’s traits and context types is necessary before customizing the higher-level renderer.

## 6. Best practices

- **Re-use pipelines** – avoid creating pipelines per frame; initialize once in `Rendered::new`.
- **Hot reload safety** – handle shader compilation errors gracefully during drag-and-drop.
- **Input normalization** – expose consistent coordinate conventions (e.g., lower-left origin for shaders).
- **Resource lifetime** – drop GPU resources explicitly when swapping scenes to avoid leaks.

## 7. Summary

`truck-platform` abstracts WGPU setup, provides the `Rendered` trait, and includes tooling such as the WGSL sandbox. It’s the foundation for all visualizers in the Truck ecosystem.

## 8. Implementation checklist

- **Initialize correctly** – configure WGPU features/swapchain formats before creating rendered objects.
- **Shader contract** – ensure custom shaders respect the sandbox’s `main_image` signature when using the WGSL sample.
- **Input handling** – forward window events to camera/controllers consistently.
- **Docs/tests** – update README and this guide when adding new viewer features.
