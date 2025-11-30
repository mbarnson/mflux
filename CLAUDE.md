# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MFLUX is a line-by-line port of FLUX, Qwen Image, and FIBO image generation models to Apple MLX (Machine Learning eXplore framework) for running on local Apple Silicon Macs. The project prioritizes minimal, explicit code for readability over generality.

**Key Philosophy:**
- Network architectures are hardcoded (no config files except tokenizers)
- Minimal dependencies: MLX, Huggingface Transformers (for tokenizers), NumPy, and Pillow
- Tiny codebase (~25,300 lines of Python) avoiding heavy abstractions

## Common Commands

```bash
# Install (developers)
make install

# Run tests
make test
pytest tests/                    # Run all tests
pytest tests/ -m "not high_memory_requirement"  # Skip memory-intensive tests

# Lint and format
make lint                        # Run Ruff linter
make format                      # Auto-format code
make check                       # Run pre-commit (lint + format)

# Build
make build                       # Build distribution packages

# Generation commands (after installation)
mflux-generate --prompt "your prompt" --model dev --steps 20
mflux-generate-controlnet       # ControlNet-guided generation
mflux-generate-depth            # Depth-guided generation
mflux-generate-fill             # Inpainting
mflux-generate-fibo             # FIBO text-to-image
```

## Architecture

### Directory Structure

- `src/mflux/models/` - AI model implementations (FLUX, Qwen, FIBO, DepthPro)
- `src/mflux/models/flux/variants/` - FLUX extensions (txt2img, controlnet, depth, fill, redux, kontext)
- `src/mflux/config/` - Configuration management (Config, ModelConfig, RuntimeConfig)
- `src/mflux/callbacks/` - Plugin system for generation pipeline
- `src/mflux/ui/cli/` - Command-line interface parsers
- `src/mflux/utils/` - Utility functions (image processing, metadata, downloads)
- `tests/` - Test suite with reference image comparisons

### Key Classes

- `Flux1` (`models/flux/variants/txt2img/flux.py`) - Main FLUX model
- `QwenImage`, `QwenImageEdit` - Qwen models
- `FIBO` - Bria FIBO model
- `FluxInitializer` - Loads weights, tokenizers, initializes submodels
- `Config` - Runtime parameters (steps, guidance, dimensions)
- `ModelConfig` - Model-specific settings

### Callback System

Callbacks provide extensibility without modifying core code:
- `BeforeLoopCallback` - Runs before generation starts
- `InLoopCallback` - Runs each denoising step
- `AfterLoopCallback` - Runs after generation
- `InterruptCallback` - Handles user interruption

Built-in: `BatterySaver`, `MemorySaver`, `StepwiseHandler`

## Code Style

- **Formatter/Linter:** Ruff (line-length: 120, indent: 4 spaces, Python 3.10+)
- **Type checking:** MyPy
- **Pre-commit hooks:** ruff-check, ruff-format, typos, mypy

**Naming Conventions:**
- Classes: PascalCase
- Functions/Methods: snake_case
- Constants: UPPER_SNAKE_CASE

## Testing

Tests use pytest with reference image comparisons:

```python
ImageGeneratorTestHelper.assert_matches_reference_image(
    reference_image_path="reference.png",
    model_class=Flux1,
    model_config=ModelConfig.schnell(),
    steps=2,
    seed=42,
    prompt="test prompt"
)
```

Reference images are stored in `tests/resources/`.

## Platform Requirements

- macOS with Apple Silicon (ARM64 only)
- Python 3.10+
- MLX framework (0.27.0 to 0.30.x)
