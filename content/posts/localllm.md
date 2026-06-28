---
title: "Local LLM – Single User vs Multiple Users"
date: 2026-06-28
summary: "How to serve a local LLM on GPU with llama.cpp for yourself, or with vllm when you need to share it with a team."
tags: ["ai", "llm", "llama.cpp", "vllm", "local"]
---

The full reference is available as a [PDF guide](/res/localllm.pdf).

Running a local LLM comes down to one question: who is going to use it?

- **Single user** → `llama.cpp`. Simpler, lower latency, sequential requests.
- **Multiple users** → `vllm`. Higher throughput via continuous batching, supports multi-GPU sharding.

## llama.cpp (single user)

Build with CUDA and OpenSSL support (OpenSSL lets the server pull models directly from Hugging Face):

```bash
git clone https://github.com/ggml-org/llama.cpp.git && cd llama.cpp
cmake -B build -DGGML_CUDA=ON -DLLAMA_OPENSSL=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j
```

Serve:

```bash
CUDA_VISIBLE_DEVICES=0 ./build/bin/llama-server \
  -hf unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q5_K_XL \
  --jinja \
  -c 65536 \
  --port 8034 \
  -ngl 999 \
  --flash-attn on
```

This exposes a web UI at `localhost:8034` and an OpenAI-compatible API at `localhost:8034/v1`. The `-hf` flag downloads the model automatically. Use `-ngl 999` to offload all layers to GPU. `--jinja` is required for tool calling.

The best model I tested is `unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q5_K_XL` — reasoning quality on par with frontier models from a few months ago.

## vllm (multiple users)

```bash
uv pip install vllm  # CUDA 12.x
uv run vllm serve Qwen/Qwen2.5-7B-Instruct \
  --host 127.0.0.1 --port 8000 \
  --max-model-len 32768 \
  --tensor-parallel-size 4
```

For a web UI, spin up Open WebUI via Docker:

```bash
docker run -d --network=host \
  -e OPENAI_API_BASE_URL=http://127.0.0.1:8000/v1 \
  -e OPENAI_API_KEY=sk-no-key \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

Access it at `localhost:8080`. If the server is remote, forward the port: `ssh -L 8080:127.0.0.1:8080 user@server`.

{{< alert >}}
I tried hard to run Qwen 3.6 on vllm and failed due to CUDA compatibility issues. For anything serious, stick with llama.cpp. CUDA compatibility across models is a madness.
{{< /alert >}}

## Connecting to Opencode

Both backends expose an OpenAI-compatible API, so [Opencode](https://opencode.ai) can use either. Add a provider block to `~/.config/opencode/opencode.json`:

```json
"qwen3.6-local": {
  "name": "llama.cpp Qwen",
  "npm": "@ai-sdk/openai-compatible",
  "options": { "baseURL": "http://127.0.0.1:8034/v1" },
  "models": {
    "unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q5_K_XL": {
      "name": "Qwen 3.6 local",
      "tool_call": true
    }
  }
}
```

## Summary

| Aspect | llama.cpp | vllm |
|---|---|---|
| Best for | Single user | Multiple users |
| Throughput | Sequential | Continuous batching |
| Setup | Low | Moderate |
| Model format | GGUF (quantised) | Full-precision HF |
| Web UI | Built-in | Open WebUI (Docker) |

Start with `llama.cpp` + Qwen 3.6 for personal use. Switch to `vllm` only when you need to share the model with teammates.
