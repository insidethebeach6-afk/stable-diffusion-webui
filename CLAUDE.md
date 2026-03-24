# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is [stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) — a Gradio + FastAPI web interface for Stable Diffusion AI image generation. The app runs as a local web server exposing both a browser UI and a REST API.

## Common Commands

**Run the application:**
```bash
./webui.sh                # macOS/Linux (handles Python venv setup automatically)
python launch.py          # Direct Python launcher
python webui.py --nowebui # API-only mode (FastAPI, no Gradio UI)
```

**Lint:**
```bash
ruff check modules/           # Python linting (configured in pyproject.toml, targets Python 3.9+)
npm run lint                  # ESLint for JavaScript files
npm run fix                   # Auto-fix JavaScript issues
```

**Test:**
```bash
pytest                        # Runs tests (base URL: http://127.0.0.1:7860 — server must be running)
```

**User configuration:** Edit `webui-user.sh` to set env vars and CLI flags before launch (do not edit `webui.sh` directly).

## Architecture

### Entry Points
- `webui.sh` → `launch.py` → `webui.py` — the normal startup chain
- `webui.py`: `webui()` starts Gradio + FastAPI; `api_only()` starts FastAPI only

### Layer Breakdown

```
Gradio Web UI  (modules/ui.py, html/, javascript/)
     ↓
FastAPI REST API  (modules/api/api.py, modules/api/models.py)
     ↓
Processing Pipeline  (modules/processing.py, modules/txt2img.py, modules/img2img.py)
     ↓
Model / Sampler Layer  (modules/sd_models.py, modules/sd_samplers.py, modules/sd_vae.py)
     ↓
PyTorch inference  (patched via modules/sd_hijack.py)
```

### Key Modules

| File | Responsibility |
|---|---|
| `modules/shared.py` | Global state shared across all modules |
| `modules/shared_options.py` | All user-configurable settings definitions |
| `modules/processing.py` | Core `StableDiffusionProcessingTxt2Img` / `Img2Img` classes |
| `modules/sd_models.py` | Checkpoint discovery, loading, switching |
| `modules/sd_hijack.py` | Monkey-patches applied to diffusion model internals |
| `modules/script_callbacks.py` | Callback hooks used by the extension system |
| `modules/scripts.py` | Script/extension registration and invocation |
| `modules/call_queue.py` | GPU call queue (prevents concurrent inference) |
| `modules/devices.py` | Device selection (CUDA / MPS / CPU) |
| `modules/api/api.py` | All FastAPI route handlers |

### Extension System

Extensions live in `extensions/` (user-installed) and `extensions-builtin/` (official). They hook into the app via:
- `modules/script_callbacks.py` — event hooks (`on_app_started`, `on_before_ui`, etc.)
- `modules/scripts.py` — script registration (adds UI tabs/sections and processing steps)

Built-in extensions include LoRA (`extensions-builtin/Lora/`), several upscalers (LDSR, ScuNET, SwinIR), canvas zoom, mobile UI, and hypertile.

### Frontend

- `modules/ui.py` — builds the entire Gradio interface
- `javascript/` — client-side logic (drag-drop, progress polling, extra networks panel, image viewer)
- `html/` — Gradio HTML templates injected into the page

### JavaScript Style
- 4-space indentation, semicolons required (enforced by `.eslintrc.js`)

### Python Style
- Ruff linter, Python 3.9+ target (configured in `pyproject.toml`)
