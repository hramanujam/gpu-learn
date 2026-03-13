# GPU Programming Learning Path — wgpu + Rust

## Context
Learn GPU programming fundamentals using Mac's Apple Silicon GPU via wgpu (Rust). Progressive series of mini-projects, each building on the last, culminating in an audio visualizer.

## Project location
`/Users/harishramanujam/projects/gpu-learn/`

## Phase 1: Hello Triangle (GPU basics)
**Goal**: Minimal wgpu app — open a window, render a colored triangle.

**Concepts learned**:
- wgpu device/surface/queue setup
- Render pipeline (vertex shader → rasterizer → fragment shader)
- WGSL shader language basics
- Vertex buffers and attributes
- The render pass / command encoder pattern

**Files**:
- `src/bin/01_triangle.rs` — window + render loop
- `shaders/triangle.wgsl` — vertex + fragment shader

**Dependencies**: `wgpu`, `winit` (windowing), `pollster` (async runtime), `env_logger` + `log` (logging)

**Logging**: All phases use `env_logger` with debug-level logging for GPU operations (device init, shader compile, buffer creation, frame timing). Run with `RUST_LOG=info` (or `debug` / `WGPU_LOG=debug` for wgpu internals).

## Phase 2: Mandelbrot Explorer (compute shaders)
**Goal**: GPU-computed Mandelbrot fractal with interactive zoom/pan.

**Concepts learned**:
- Compute shaders and workgroups
- Storage buffers and bind groups
- GPU → texture → screen pipeline
- Uniforms (zoom level, center coordinates)
- Interactive input handling

**Files**:
- `src/bin/02_mandelbrot.rs` — window + compute dispatch + render
- `shaders/mandelbrot.wgsl` — compute shader for fractal + render shaders

## Phase 3: Audio Visualizer (real-time data streaming)
**Goal**: Capture system audio, compute FFT, render frequency spectrum and waveform in real-time.

**Concepts learned**:
- Streaming CPU data to GPU each frame (dynamic buffers)
- Multiple render passes or draw calls
- Color mapping and visual effects in shaders
- Real-time performance considerations
- Audio capture on macOS (via `cpal` crate)

**Files**:
- `src/bin/03_visualizer.rs` — audio capture + FFT + GPU render loop
- `shaders/visualizer.wgsl` — bar/waveform rendering shaders

**Dependencies** (additional): `cpal` (audio capture), `rustfft` (FFT)

**Note**: macOS system audio capture requires a virtual audio device (e.g., BlackHole) or we capture microphone input instead. We'll decide when we get there.

## Shared structure
```
gpu-learn/
├── Cargo.toml
├── shaders/
│   ├── triangle.wgsl
│   ├── mandelbrot.wgsl
│   └── visualizer.wgsl
├── src/
│   └── bin/
│       ├── 01_triangle.rs
│       ├── 02_mandelbrot.rs
│       └── 03_visualizer.rs
├── tests/
│   ├── common/
│   │   └── mod.rs              # Shared GPU test helpers (headless adapter/device)
│   ├── 01_triangle.rs
│   ├── 02_mandelbrot.rs
│   └── 03_visualizer.rs
└── explanations/
    ├── 01_triangle.md      # GPU basics, wgpu setup, shader intro
    ├── 02_mandelbrot.md     # Compute shaders, bind groups, uniforms
    └── 03_visualizer.md     # Real-time streaming, audio capture, FFT
```

Each phase is a standalone binary (`cargo run --bin 01_triangle`).
Each phase includes an `explanation.md` with GPU concepts, Rust notes, and Q&A accumulated during the session.

## Development approach: Red/Green TDD
For each phase, **write tests first**, then implement until tests pass.

**Test scope** — two layers:
1. **Unit tests** (logic only, no GPU): vertex data generation, coordinate transforms, color mapping, FFT math
2. **Integration tests** (headless GPU): device/adapter creation, shader compilation, buffer creation, compute dispatch — using wgpu without a window (request adapter with `compatible_surface: None`)

**TDD cycle per feature**:
1. Write a failing test (`cargo test` → red)
2. Implement the minimum code to pass (`cargo test` → green)
3. Refactor if needed, re-run tests

**Test files**: `tests/` directory at project root, one file per phase:
- `tests/01_triangle.rs`
- `tests/02_mandelbrot.rs`
- `tests/03_visualizer.rs`

Shared GPU test helpers (adapter/device setup) go in `tests/common/mod.rs`.

## Audio capture (Phase 3)
Use **BlackHole** virtual audio device to capture system audio (Apple Music, etc.) as clean digital input.

## Verification
- Phase 1: Window opens, colored triangle renders, resizing works
- Phase 2: Fractal renders, mouse scroll zooms, click-drag pans
- Phase 3: Audio input produces real-time bars/waveform in the window
