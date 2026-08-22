# vLLM ROCm Docker for RX 7900 XTX

Custom Docker image for running vLLM on AMD RX 7900 XTX (gfx1100) with ROCm 6.2.

## Features

- **vLLM v0.6.5** - High-throughput LLM inference engine
- **ROCm 6.2** - AMD GPU support
- **Triton Flash Attention** - Default attention backend for gfx1100
- **Qwen3.5-4B Ready** - Optimized for consumer AMD GPUs

## Quick Start

### Pull from Docker Hub

```bash
docker pull kapteng/vllm-rocm:latest
```

### Run Container

```bash
docker run -it \
  --name vllm-dev \
  --network=host \
  --group-add=video \
  --ipc=host \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  --device /dev/kfd \
  --device /dev/dri \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  -e HF_HOME=/root/.cache/huggingface \
  -e HSA_OVERRIDE_GFX_VERSION=11.0.0 \
  -e PYTORCH_ROCM_ARCH=gfx1100 \
  kapteng/vllm-rocm:latest
```

### Start vLLM Server

```bash
# Inside container
vllm serve Qwen/Qwen3.5-4B \
  --host 0.0.0.0 \
  --port 8000 \
  --max-model-len 32768 \
  --reasoning-parser qwen3
```

## Build Locally

### Prerequisites

- Docker with BuildKit
- ROCm compatible GPU (RX 7900 XTX)

### Build

```bash
git clone https://github.com/syofyan/vllm-rocm-docker.git
cd vllm-rocm-docker

DOCKER_BUILDKIT=1 docker build \
  --build-arg BUILD_FA="0" \
  --build-arg BUILD_TRITON="1" \
  --build-arg PYTORCH_ROCM_ARCH="gfx1100" \
  -f Dockerfile.rocm \
  -t kapteng/vllm-rocm:latest \
  .
```

## GitHub Actions (Automated Build)

This repository includes GitHub Actions workflow for automated Docker builds.

### Setup

1. **Create GitHub Repository**
   ```bash
   git init
   git remote add origin https://github.com/syofyan/vllm-rocm-docker.git
   ```

2. **Add Secrets in GitHub**
   Go to: Settings → Secrets and variables → Actions → New repository secret

   | Secret Name | Value |
   |-------------|-------|
   | `DOCKERHUB_USERNAME` | `kapteng` |
   | `DOCKERHUB_TOKEN` | Docker Hub Access Token |

3. **Get Docker Hub Access Token**
   - Login to [Docker Hub](https://hub.docker.com)
   - Go to: Account Settings → Security → Access Tokens
   - Create new token with Read & Write permissions
   - Copy the token

4. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Initial commit: vLLM ROCm Docker for RX 7900 XTX"
   git branch -M main
   git push -u origin main
   ```

5. **Trigger Build**
   - Push to `main` branch → automatic build
   - Push tag `v*` → builds with tag version
   - Manual trigger → Actions tab → Run workflow

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `HSA_OVERRIDE_GFX_VERSION` | `11.0.0` | Required for gfx1100 |
| `PYTORCH_ROCM_ARCH` | `gfx1100` | Target GPU architecture |
| `VLLM_USE_TRITON_FLASH_ATTN` | `1` | Use Triton FA (default for gfx1100) |

## Troubleshooting

### GPU Not Detected
```bash
# Ensure devices are mounted
ls -la /dev/kfd /dev/dri
```

### OOM Error
```bash
# Reduce max model length
vllm serve Qwen/Qwen3.5-4B --max-model-len 16384
```

### Hang/Freeze
```bash
# Disable HIP graphs
vllm serve Qwen/Qwen3.5-4B --enforce-eager
```

## Model Compatibility

| Model | Size | Status |
|-------|------|--------|
| Qwen/Qwen3.5-4B | 4B | ✅ Recommended |
| Qwen/Qwen3.5-2B | 2B | ✅ Supported |
| Qwen/Qwen3.5-9B | 9B | ⚠️ Tight on 24GB |
| Llama-3.1-8B | 8B | ✅ Supported |

## License

Apache 2.0

## Resources

- [vLLM Documentation](https://docs.vllm.ai)
- [AMD ROCm](https://rocm.docs.amd.com)
- [Qwen3.5 Model](https://huggingface.co/Qwen/Qwen3.5-4B)
