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

## Using uv for Package Management

The Docker image includes [uv](https://github.com/astral-sh/uv), a fast Python package installer and resolver. To add dependencies:

1. Add dependencies to `pyproject.toml[.cuda'11|.cuda12]`:
   ```toml
   dependencies = ["requests", "numpy"]
   ```

