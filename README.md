# PyMOL GPU Ray 0.1.7

A drop-in PyMOL plugin that renders the **currently visible PyMOL scene** with
Blender Cycles using **NVIDIA OptiX**, **CUDA**, or CPU fallback.

The plugin does not patch PyMOL and does not install CUDA/Python packages into
PyMOL. PyMOL exports its visible scene to COLLADA, and Blender runs as an
isolated background renderer. Technically your scene is adapted, but with care to keep some fidelity.

## Installation
1. Install [Blender 4.2 or newer](https://www.blender.org/download/).
2. Download the plugin .zip package. Do not unzip the plugin. In PyMOL, open **Plugin → Plugin Manager → Install New Plugin** and select
   `pymol-gpu-ray-0.1.7-public.zip`.
3. Restart PyMOL, then open **Plugin → GPU Ray Tracer**.
4. Click **Render**. If Blender isn't detected, select it with **Browse**.

The PNG uses your viewer's dimensions and a transparent background by default.

## Basic commands

- `gpu_ray` — render the current view.
- `gpu_ray_gui` — open the plugin window.
- `gpu_ray_status` — check render progress.
- `gpu_ray_diagnose` — check Blender and GPU setup.
