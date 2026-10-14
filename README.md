# PyMOL GPU Ray 0.1.3

A drop-in PyMOL plugin that renders the **currently visible PyMOL scene** with
Blender Cycles using **NVIDIA OptiX**, **CUDA**, or CPU fallback.

The plugin does not patch PyMOL and does not install CUDA/Python packages into
PyMOL. PyMOL exports its visible scene to COLLADA, and Blender runs as an
isolated background renderer.

## What works in this first release

- `gpu_ray` command from the PyMOL command line
- compact Qt GUI under **Plugin → GPU Ray Tracer**
- automatic Blender discovery on Windows, macOS and Linux
- `AUTO` backend selection: OptiX → CUDA → CPU fallback
- explicit `OPTIX`, `CUDA`, or `CPU` modes
- current PyMOL camera and projection through the exported COLLADA camera
- cartoons, sticks, spheres, surfaces and other geometry represented by the
  PyMOL COLLADA exporter
- PyMOL per-vertex colors and smooth normals
- PyMOL object/representation transparency exported through COLLADA
- PyMOL background color or transparent PNG output
- Cycles denoising
- asynchronous renders so PyMOL remains usable
- job status command and useful failure diagnostics
- Blender 4.x legacy COLLADA importer when available
- bundled PyMOL-specific COLLADA reader for Blender releases without COLLADA
  import support (including Blender 5.x)

## Requirements

1. **PyMOL** with `cmd.get_collada()` support. Most normal current builds have
   it. If your build lacks COLLADA/libxml2 support, the plugin reports this
   cleanly.
2. **Blender 4.2 or newer**. Blender is deliberately external to PyMOL.
3. For GPU rendering, a Blender-supported NVIDIA GPU/driver. OptiX is preferred
   on NVIDIA RTX hardware; CUDA is the compatibility fallback.

No PyPI packages are required inside PyMOL.

## Install

1. Install Blender if it is not already installed.
2. In PyMOL open **Plugin → Plugin Manager → Install New Plugin**.
3. Select `pymol-gpu-ray-0.1.3-public.zip`.
4. Open **Plugin → GPU Ray Tracer**.
5. Click **Diagnose** once. If Blender was not found automatically, select
   `blender.exe` (Windows) or the Blender executable manually.
6. Click **Render**.

The selected Blender path is remembered in `~/.pymol_gpu_ray/config.json`.
You can also set `PYMOL_GPU_RAY_BLENDER` to the full Blender executable path.

## Commands

Basic render:

```text
gpu_ray
```

4K, 128 samples:

```text
gpu_ray 3840, 2160, 128
```

Force OptiX and use a transparent background:

```text
gpu_ray backend=OPTIX, transparent=1
```

High-quality blocking render to a chosen file:

```text
gpu_ray width=3840, height=2160, samples=256, output=figure.png, wait=1
```

Check hardware/backend without rendering:

```text
gpu_ray_diagnose
```

Force a diagnostic backend:

```text
gpu_ray_diagnose OPTIX
```

Check current/background render jobs:

```text
gpu_ray_status
```

or:

```text
gpu_ray_status a1b2c3d4
```

Open the GUI from the command line:

```text
gpu_ray_gui
```

## `gpu_ray` arguments

| Argument | Default | Meaning |
| --- | ---: | --- |
| `width` | 1920 | Output width in pixels |
| `height` | 1080 | Output height in pixels |
| `samples` | 64 | Cycles path-tracing samples |
| `backend` | `AUTO` | `AUTO`, `OPTIX`, `CUDA`, or `CPU` |
| `denoise` | 1 | Enable Cycles denoising |
| `transparent` | 0 | Transparent PNG background |
| `output` | timestamped PNG | Output path |
| `blender` | auto | Explicit Blender executable |
| `open_image` | 1 | Open the finished PNG |
| `wait` | 0 | Block PyMOL until render completes |
| `bounces` | 6 | Maximum Cycles path bounces |

## Architecture

```text
PyMOL scene
    │
    ├─ cmd.get_collada()
    │   camera + visible geometry + normals + colors + transparency
    │
    ▼
temporary scene.dae + job.json
    │
    ▼
Blender background process
    │
    ├─ Blender 4.x: legacy COLLADA importer when present
    └─ Blender 5.x: bundled PyMOL COLLADA parser
    │
    ▼
Cycles
    ├─ OptiX (preferred NVIDIA path)
    ├─ CUDA
    └─ CPU fallback
    │
    ▼
PNG
```

Keeping Blender in another process is intentional: PyMOL's bundled Python,
Blender's Python, CUDA libraries, and OptiX runtime never have to coexist in
one interpreter.

## Fidelity and known limits

This is an MVP intended to be useful rather than merely demonstrate a CUDA
kernel. There are still fidelity differences from PyMOL's built-in `ray`:

- lighting uses a neutral three-area-light Cycles rig instead of reproducing
  every PyMOL light setting exactly;
- material roughness/specularity are currently normalized for a clean
  molecular-graphics look rather than mapped setting-for-setting;
- labels/text and any primitive not emitted by PyMOL's COLLADA exporter cannot
  be recovered by the plugin;
- molecular spheres/cylinders arrive tessellated from PyMOL in v0.1.0. A later
  native-primitive path can substantially reduce memory use for huge scenes;
- Blender 5.x uses the bundled fallback importer, which is deliberately scoped
  to the COLLADA subset PyMOL emits.

On a render failure, temporary scene/job files are intentionally retained and
their directory is printed in the PyMOL console. On success they are deleted.

## Troubleshooting

### Blender not found

Use the GUI Browse button or:

```text
gpu_ray blender=C:\\Program Files\\Blender Foundation\\Blender 4.5\\blender.exe
```

You can permanently point the plugin at Blender with the environment variable
`PYMOL_GPU_RAY_BLENDER`.

### AUTO uses CPU

Run:

```text
gpu_ray_diagnose
```

If OptiX and CUDA are unavailable to Blender, AUTO deliberately falls back to
CPU and prints a warning. To make missing GPU support a hard error, request an
explicit backend:

```text
gpu_ray_diagnose OPTIX
```

### PyMOL says `get_collada` is unavailable

That PyMOL build was compiled without COLLADA support. Use a normal current
PyMOL distribution or rebuild PyMOL with the required XML support.

## Development tests

The source tree includes dependency-free tests for the PyMOL-side integration
and the bundled COLLADA parser:

```bash
python -m unittest discover -s tests -v
```

The Blender backend requires Blender's `bpy` module and therefore is syntax-
checked outside Blender but integration-tested only on a machine with Blender.


## New in 0.1.2

- Brighter default look.
- Color look controls: brightness, contrast, saturation, exposure.
- Improved handling of sphere/cartoon shading in the Blender-5 fallback importer.
