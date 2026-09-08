---
title: llm-local-tool
tags:
  - llm
  - tool
  - ui
type: reference
priority: 2
finished: true
created_date: 2026-08-29
---

# LLM Local Tool

## Abstract

UI/tool layer for running local LLMs, centered on **Open WebUI** — a self-hosted web interface (ChatGPT-style) that connects to local engines like Ollama/llama.cpp or remote APIs (OpenAI-compatible). The engine layer itself is covered separately in [[LLMS/local-llm-engine]].

## Layering

```
[ Browser ] ── [ Open WebUI :3000 ] ── [ Engine (Ollama :11434 / OpenAI API) ]
```

- **Engine** (Ollama, llama.cpp, vLLM): model inference — see [[LLMS/local-llm-engine]]
- **Tool/UI** (Open WebUI): chat interface, model management, RAG, multi-user auth
- Open WebUI talks to engines over HTTP; it does not run models itself

## Official Installation (Docker)

Verified against docs.openwebui.com Quick Start (2026-08-29).

### 1. Pull the image

```bash
docker pull ghcr.io/open-webui/open-webui:main
```

Identical image also published to Docker Hub as `openwebui/open-webui`.

### 2. Run the container

```bash
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

| Flag | Purpose |
|------|---------|
| `-p 3000:8080` | Exposes the UI on port 3000 of the host |
| `-v open-webui:/app/backend/data` | Persistent storage — **crucial**, prevents DB loss between restarts |
| `--restart always` | Auto-restart on reboot/failure |

Visit http://localhost:3000 after startup. First account created becomes admin.

> [!warning] Never omit the volume
> The official docs warn: without `-v open-webui:/app/backend/data` the database is not mounted and data is lost on container restart.

## Command Variants

### Ollama on the same machine

Container cannot reach the host's Ollama (`:11434`) by default — add the host gateway:

```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Ollama on a different server

```bash
docker run -d -p 3000:8080 \
  -e OLLAMA_BASE_URL=https://example.com \
  -v open-webui:/app/backend/data \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```

### OpenAI API only (no local engine)

```bash
docker run -d -p 3000:8080 \
  -e OPENAI_API_KEY=your_secret_key \
  -v open-webui:/app/backend/data \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Bundled engine image (`:ollama` tag)

Ships Ollama inside the same container:

```bash
# GPU (Nvidia)
docker run -d -p 3000:8080 --gpus=all \
  -v ollama:/root/.ollama -v open-webui:/app/backend/data \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:ollama

# CPU only: same command without --gpus=all
```

Alternative for GPU with separate Ollama: `:cuda` tag + `--gpus all --add-host=host.docker.internal:host-gateway`.

## Gotchas

- **First boot is slow** — on startup Open WebUI auto-downloads the default RAG embedding model (`sentence-transformers/all-MiniLM-L6-v2`, ~900MB incl. ONNX/OpenVINO variants) from Hugging Face. The UI is unresponsive until this finishes; subsequent restarts are fast since it's cached in the volume. There is no official "disable RAG" switch — deleting the model just re-downloads it. Block via `HF_HUB_OFFLINE=1` (+ `RAG_EMBEDDING_MODEL_AUTO_UPDATE=false`), or offload with `RAG_EMBEDDING_ENGINE=ollama|openai`. A second small model (`TaylorAI/bge-micro-v2`) is for the evaluations feature.
- **`:network=host` changes the port** — with `--network=host` the `-p` mapping is ignored and the UI is at `http://localhost:8080` directly. Used when the container still cannot reach Ollama at the host IP.
- **`:dev` tag needs its own volume** — dev builds may include DB migrations that a release image cannot read back. Use `-v open-webui-dev:/app/backend/data --name open-webui-dev` and a different port (e.g. 3001); never share the volume with a `:main` instance.

## Docker Compose Variant

```yaml
services:
  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    ports:
      - "3000:8080"
    volumes:
      - open-webui:/app/backend/data

volumes:
  open-webui:
```

See [[Docker/docker-compose]] for compose fundamentals and [[Docker/docker-basic-commands]] for the underlying `docker run` syntax used above.

## Sources

- [Open WebUI Quick Start](https://docs.openwebui.com/getting-started/quick-start/)
- [open-webui GitHub README](https://github.com/open-webui/open-webui)
