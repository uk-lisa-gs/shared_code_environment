# Use Python 3.12 slim image as base
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Install git (required for git+ dependencies when installing packages)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    build-essential \
    python3-dev \
    libssl-dev \
    pkg-config \
    ca-certificates \
  && rm -rf /var/lib/apt/lists/*

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Copy project files
COPY pyproject.toml ./
COPY README.md ./

# Verify uv is installed
RUN uv --version

# Install dependencies from pyproject.toml  
# Set GIT_SSL_NO_VERIFY as a workaround for SSL certificate verification issues
ENV GIT_SSL_NO_VERIFY=1
RUN uv pip install --system .
ENV GIT_SSL_NO_VERIFY=

# Set the default command to show Python and uv versions
CMD ["sh", "-c", "python --version && uv --version"]
