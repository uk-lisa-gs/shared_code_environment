# shared_code_environment

Github CI to build a shared code environment for UK LISA work and documentation related to this.

## Features

This repository includes:

- **Docker Images**: Python 3.12 Docker images with `uv` package manager pre-installed in three variants:
  - **CPU-only**: Lightweight image for CPU-based computation
  - **CUDA 11**: GPU-enabled image with CUDA 11.8 support
  - **CUDA 12**: GPU-enabled image with CUDA 12.1 support
- **GitHub CI Workflow**: Automated Docker image build and push to GitHub Container Registry (GHCR) on push/PR to main and develop branches
- **Python Project Configuration**: Basic `pyproject.toml` setup for Python projects

## Using the Pre-built Docker Images

The Docker images are automatically built and pushed to GitHub Container Registry. You can pull and use them directly:

### CPU-only Image

```bash
# Pull the latest CPU image from main branch
docker pull ghcr.io/uk-lisa-gs/shared_code_environment:main-cpu

# Run the image
docker run --rm ghcr.io/uk-lisa-gs/shared_code_environment:main-cpu

# Use it as a base for your own work
docker run -it --rm -v $(pwd):/workspace ghcr.io/uk-lisa-gs/shared_code_environment:main-cpu bash
```

### CUDA 11 Image (GPU)

```bash
# Pull the latest CUDA 11 image from main branch
docker pull ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda11

# Run the image with GPU support
docker run --rm --gpus all ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda11

# Use it as a base for your own work
docker run -it --rm --gpus all -v $(pwd):/workspace ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda11 bash
```

### CUDA 12 Image (GPU)

```bash
# Pull the latest CUDA 12 image from main branch
docker pull ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda12

# Run the image with GPU support
docker run --rm --gpus all ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda12

# Use it as a base for your own work
docker run -it --rm --gpus all -v $(pwd):/workspace ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda12 bash
```

### Available Tags

Each variant has the following tag patterns:
- `<branch>-<variant>` - Latest build from a branch (e.g., `main-cpu`, `main-cuda11`, `main-cuda12`)
- `<branch>-<sha>-<variant>` - Specific commit builds (e.g., `main-abc1234-cpu`)
- `latest-<variant>` - Latest build from the main branch (e.g., `latest-cpu`, `latest-cuda11`, `latest-cuda12`)

Where `<variant>` is one of: `cpu`, `cuda11`, or `cuda12`

## Building the Docker Images

### Locally

```bash
# Build CPU-only image
docker build -f Dockerfile.cpu -t shared-code-environment:cpu .
docker run --rm shared-code-environment:cpu

# Build CUDA 11 image
docker build -f Dockerfile.cuda11 -t shared-code-environment:cuda11 .
docker run --rm shared-code-environment:cuda11

# Build CUDA 12 image
docker build -f Dockerfile.cuda12 -t shared-code-environment:cuda12 .
docker run --rm shared-code-environment:cuda12
```

### Via GitHub Actions

The Docker images are automatically built by GitHub Actions on:
- Push to `main` or `develop` branches
- Pull requests to `main` or `develop` branches
- Manual workflow dispatch

All three variants (CPU, CUDA 11, CUDA 12) are built in parallel for each trigger.

## GPU Support

### Current State
The CUDA 11 and CUDA 12 images currently install the **CPU versions** of all dependencies. This allows the images to be built and tagged correctly while maintaining compatibility with the existing codebase.

### Recommended Changes for Full GPU Support

To enable full GPU acceleration, the following changes would be needed:

#### 1. PyTorch (torch)
Currently using CPU-only wheels. For GPU support:

**CUDA 11.8:**
```toml
[[tool.uv.index]]
name = "pytorch-cu118"
url = "https://download.pytorch.org/whl/cu118"
explicit = true

[tool.uv.sources]
torch = [
    { index = "pytorch-cu118" },
]
```

**CUDA 12.1:**
```toml
[[tool.uv.index]]
name = "pytorch-cu121"
url = "https://download.pytorch.org/whl/cu121"
explicit = true

[tool.uv.sources]
torch = [
    { index = "pytorch-cu121" },
]
```

#### 2. PyCBC (pycbc)
Has GPU support through PyCUDA/CuPy. Add to dependencies:
- For CUDA 11: `cupy-cuda11x`
- For CUDA 12: `cupy-cuda12x`

#### 3. gbgpu
Already GPU-capable. May benefit from CUDA-specific compilation flags set via environment variables during build.

#### 4. Other Packages
Most other packages (numpy, scipy, astropy, etc.) will automatically benefit from GPU acceleration when PyTorch and/or CuPy are properly configured, as they can delegate operations to GPU arrays.

### Creating GPU-Specific Configurations

To create separate GPU configurations, you would need:
1. Separate `pyproject.toml` files (e.g., `pyproject.cuda11.toml`, `pyproject.cuda12.toml`)
2. Modify Dockerfiles to copy and use the appropriate configuration
3. Adjust the build to use `uv pip install` with the GPU-specific torch wheels

## Using uv for Package Management

The Docker image includes [uv](https://github.com/astral-sh/uv), a fast Python package installer and resolver. To add dependencies:

1. Add dependencies to `pyproject.toml`:
   ```toml
   dependencies = ["requests", "numpy"]
   ```

2. Update the Dockerfile to install dependencies:
   ```dockerfile
   RUN uv pip install --system .
   ```

## Development

To extend this project with your own Python code:

1. Add your Python code to a `src/` directory
2. Update `pyproject.toml` with your dependencies
3. Modify the Dockerfile to install your package
4. Push changes to trigger the CI build

