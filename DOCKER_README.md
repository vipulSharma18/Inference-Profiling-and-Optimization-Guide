# Docker Setup for Inference Profiling Puzzles

This repository contains three separate packages, each with its own Docker container for isolated development and deployment.

## Packages

1. **gemlite_autotune** - Gemlite autotuning experiments
2. **torchao_float8** - TorchAO FP8 benchmarking
3. **torchinductor_cudagraph_memory** - TorchInductor CUDA graph memory profiling

## Prerequisites

- Docker (with NVIDIA Container Toolkit for GPU support)
- Docker Buildx (for advanced caching features)

## Building Docker Images

### Build All Images from Repository Root

```bash
# Build gemlite_autotune
docker build -f gemlite_autotune/Dockerfile -t gemlite-autotune:latest .

# Build torchao_float8
docker build -f torchao_float8/Dockerfile -t torchao-float8:latest .

# Build torchinductor_cudagraph_memory
docker build -f torchinductor_cudagraph_memory/Dockerfile -t torchinductor-cudagraph-memory:latest .
```

### Using Docker Compose (if needed)

Create a `docker-compose.yml` for easier management:

```yaml
version: '3.8'

services:
  gemlite-autotune:
    build:
      context: .
      dockerfile: gemlite_autotune/Dockerfile
    image: gemlite-autotune:latest
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all

  torchao-float8:
    build:
      context: .
      dockerfile: torchao_float8/Dockerfile
    image: torchao-float8:latest
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all

  torchinductor-cudagraph-memory:
    build:
      context: .
      dockerfile: torchinductor_cudagraph_memory/Dockerfile
    image: torchinductor-cudagraph-memory:latest
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
```

## Running Containers

### With GPU Support

```bash
# Run gemlite_autotune
docker run --gpus all -it --rm gemlite-autotune:latest

# Run torchao_float8
docker run --gpus all -it --rm torchao-float8:latest

# Run torchinductor_cudagraph_memory
docker run --gpus all -it --rm torchinductor-cudagraph-memory:latest
```

### Mount Local Code for Development

```bash
docker run --gpus all -it --rm \
  -v $(pwd)/gemlite_autotune:/workspace/gemlite_autotune \
  gemlite-autotune:latest
```

## CI/CD with GitHub Actions

The repository includes three separate GitHub Actions workflows:

1. `.github/workflows/build-torchao-float8.yml`
2. `.github/workflows/build-gemlite-autotune.yml`
3. `.github/workflows/build-torchinductor-cudagraph.yml`

Each workflow:
- **Builds its specific Docker image** independently
- **Only runs when relevant files change** (path filtering)
- **Caches Docker layers** and uv dependencies for faster builds
- **Tests the image** to ensure it's working correctly
- **Cleans up resources** after the build

### Key Features

1. **Individual Status Badges**: Each workflow has its own status badge in the README
2. **Path Filtering**: Workflows only trigger when their package or common_utils changes
3. **Layer Caching**: Leverages Docker BuildKit for efficient layer caching
4. **uv Caching**: Caches Python dependencies managed by uv
5. **Resource Management**: Automatic cleanup after each build

### Triggering the Workflows

Each workflow runs on:
- Push to `main` branch (only when relevant files change)
- Pull requests to `main` branch (only when relevant files change)
- Manual trigger via GitHub Actions UI (workflow_dispatch)

## Docker Image Features

Each Docker image includes:

- **NVIDIA CUDA 12.8** with cuDNN 9
- **Python 3.12**
- **uv package manager** for fast dependency installation
- **All project dependencies** from `pyproject.toml`
- **Common utilities** from the shared `common_utils` package

## Optimizations

### Build Caching

The Dockerfiles are optimized for layer caching:
1. System dependencies are installed first (rarely changes)
2. `pyproject.toml` and `uv.lock` are copied before the full codebase
3. Dependencies are installed with `uv sync --frozen`
4. Application code is copied last

### uv Benefits

- **Fast dependency resolution** and installation
- **Reproducible builds** with lock files
- **Efficient caching** of downloaded packages

## Troubleshooting

### GPU Not Available in Container

Ensure you have:
1. NVIDIA drivers installed on host
2. NVIDIA Container Toolkit installed
3. Use `--gpus all` flag when running containers

### Out of Disk Space During Build

```bash
# Clean up Docker resources
docker system prune -af --volumes

# Remove unused images
docker image prune -a
```

### Build Failing on GitHub Actions

Check:
1. Runner has enough disk space (workflow includes cleanup steps)
2. Lock files are committed to repository
3. All dependencies are available

## Development Workflow

1. Make changes to code locally
2. Test in Docker container with mounted volumes
3. Commit changes (including updated lock files if dependencies changed)
4. Push to GitHub - CI will build and test the affected package(s) automatically
   - Only the workflow for the changed package will run (thanks to path filtering)
   - If `common_utils` changes, all three workflows will run

## Additional Notes

- All three packages depend on `common_utils` which is copied into each container
- PyTorch is installed from the CUDA 12.8 wheel repository
- Lock files ensure reproducible builds across environments

