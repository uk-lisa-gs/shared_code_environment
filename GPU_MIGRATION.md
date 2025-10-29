# GPU Support Migration Guide

This document details the specific changes needed to enable GPU support in the Docker images.

## Overview

The repository currently builds three Docker image variants:
- **CPU-only** (`Dockerfile.cpu`): Uses CPU versions of all packages
- **CUDA 11** (`Dockerfile.cuda11`): Uses CUDA 11.8 base image but installs CPU packages
- **CUDA 12** (`Dockerfile.cuda12`): Uses CUDA 12.1 base image but installs CPU packages

To enable full GPU support, you need to create variant-specific configurations.

## Required Changes by Package

### 1. PyTorch (torch)

**Current Configuration** (in `pyproject.toml`):
```toml
[tool.uv.sources]
torch = [
    { index = "pytorch-cpu" },
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true
```

**For CUDA 11.8** (create `pyproject.cuda11.toml`):
```toml
[tool.uv.sources]
torch = [
    { index = "pytorch-cu118" },
]

[[tool.uv.index]]
name = "pytorch-cu118"
url = "https://download.pytorch.org/whl/cu118"
explicit = true
```

**For CUDA 12.1** (create `pyproject.cuda12.toml`):
```toml
[tool.uv.sources]
torch = [
    { index = "pytorch-cu121" },
]

[[tool.uv.index]]
name = "pytorch-cu121"
url = "https://download.pytorch.org/whl/cu121"
explicit = true
```

### 2. PyCBC (pycbc)

PyCBC has GPU support through CuPy. Add to dependencies:

**CUDA 11:**
```toml
dependencies = [
    # ... existing dependencies ...
    "cupy-cuda11x",
]
```

**CUDA 12:**
```toml
dependencies = [
    # ... existing dependencies ...
    "cupy-cuda12x",
]
```

### 3. gbgpu

This package is already GPU-capable. No changes to dependencies needed, but you may need to set environment variables during build:

```dockerfile
ENV CUDA_HOME=/usr/local/cuda
ENV PATH=${CUDA_HOME}/bin:${PATH}
ENV LD_LIBRARY_PATH=${CUDA_HOME}/lib64:${LD_LIBRARY_PATH}
```

### 4. Other Packages

The following packages will automatically benefit from GPU support once PyTorch and CuPy are GPU-enabled:
- **numpy**: Operations can be accelerated via CuPy
- **scipy**: Some operations benefit from GPU-enabled libraries
- **fastemriwaveforms**: Likely benefits from PyTorch GPU support
- **fastlisaresponse**: Likely benefits from PyTorch GPU support
- **eryn**: MCMC sampler that can leverage GPU through PyTorch

No explicit changes needed for these packages.

## Implementation Strategy

### Option 1: Separate Configuration Files

Create three configuration files:
- `pyproject.toml` (CPU - existing)
- `pyproject.cuda11.toml` (CUDA 11)
- `pyproject.cuda12.toml` (CUDA 12)

Update Dockerfiles:
```dockerfile
# In Dockerfile.cuda11
COPY pyproject.cuda11.toml pyproject.toml
COPY README.md ./
RUN uv pip install --system .
```

### Option 2: Build Arguments

Use build arguments to conditionally modify the configuration:

```dockerfile
ARG CUDA_VERSION=cpu
COPY pyproject.toml ./
RUN if [ "$CUDA_VERSION" = "cu118" ]; then \
      sed -i 's/pytorch-cpu/pytorch-cu118/g' pyproject.toml && \
      sed -i 's|https://download.pytorch.org/whl/cpu|https://download.pytorch.org/whl/cu118|g' pyproject.toml; \
    fi
```

### Option 3: Runtime Installation

Install GPU packages after the base installation:

```dockerfile
RUN uv pip install --system .
RUN uv pip uninstall torch && \
    uv pip install torch --index-url https://download.pytorch.org/whl/cu118
RUN uv pip install cupy-cuda11x
```

## Recommended Approach

**Option 1** (Separate Configuration Files) is recommended because:
- Clear separation of concerns
- Easy to maintain and test
- No risk of sed/regex errors
- Explicit about what's installed in each image

## Testing GPU Support

After implementing changes, test GPU support with:

```bash
# Test PyTorch GPU
docker run --gpus all ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda12 \
    python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}'); print(f'CUDA devices: {torch.cuda.device_count()}')"

# Test CuPy
docker run --gpus all ghcr.io/uk-lisa-gs/shared_code_environment:main-cuda12 \
    python -c "import cupy as cp; print(f'CuPy device: {cp.cuda.Device()}'); print(cp.array([1, 2, 3]))"
```

## Migration Checklist

- [ ] Create `pyproject.cuda11.toml` with CUDA 11.8 PyTorch index
- [ ] Create `pyproject.cuda12.toml` with CUDA 12.1 PyTorch index
- [ ] Add cupy-cuda11x to CUDA 11 dependencies
- [ ] Add cupy-cuda12x to CUDA 12 dependencies
- [ ] Update Dockerfile.cuda11 to use pyproject.cuda11.toml
- [ ] Update Dockerfile.cuda12 to use pyproject.cuda12.toml
- [ ] Add CUDA environment variables to GPU Dockerfiles
- [ ] Test builds locally
- [ ] Test GPU functionality with actual GPU hardware
- [ ] Update documentation with performance comparisons
