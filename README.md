# pod-jupyter-ml

The `jupyter-ml` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships a GPU JupyterLab that also serves a CRDT MCP
endpoint, composing the llama.cpp + vLLM/unsloth ML stack.

## What it provides

A Tier-2 GPU meta-layer that serves JupyterLab on port `8888` from the
`/workspace` volume under supervisord and exposes a CRDT MCP endpoint at `/mcp`
for agent-driven notebook editing. It composes the `llama-cpp` (llama.cpp CLI +
GGUF tools), `unsloth` (vLLM inference + LoRA fine-tuning), and `jupyter-mcp`
candies on top of the spaCy NLP model and the `jupyterlab-quarto` extension.

| Property | Value |
|---|---|
| Service | `jupyter-ml` (`jupyter lab --ip=0.0.0.0 --port=8888`, `restart: always`) |
| Port | `8888` (JupyterLab HTTP + MCP at `/mcp`) |
| Requires | `layer-cuda`, `layer-supervisord`, `plugin-mcp` (the out-of-process `mcp:` check verb) |
| Candy | `layer-llama-cpp`, `layer-unsloth`, `layer-jupyter-mcp` |
| Volume | `workspace` at `/workspace` |
| Env | `NVIDIA_PYTHON_PROJECT=~/.pixi`, `LD_LIBRARY_PATH=/usr/lib64:$HOME/llama.cpp` |
| mcp_provide | `jupyter` at `http://{{.ContainerName}}:8888/mcp` (http transport) |

Every composed piece lands a concrete, probeable artifact — the `llama-cli`
binary, the vLLM wheel in the pixi env, the spaCy model, the quarto
labextension, and the live notebook + MCP API — so being wrong is observable.

## How to use it

```bash
charly box build jupyter-ml
charly config jupyter-ml
charly start jupyter-ml
charly status jupyter-ml
charly logs jupyter-ml -f
# JupyterLab:  http://localhost:8888
# MCP endpoint: http://localhost:8888/mcp
```

The MCP server is reachable at `http://localhost:8888/mcp` (Streamable HTTP).
The MCP tools cover notebook management (`notebook_*`) and CRDT cell operations
(`cell_*`); the server manages CRDT rooms invisibly.

## Verify

```bash
charly shell jupyter-ml -c "pixi run verify-pytorch"
charly shell jupyter-ml -c "pixi run verify-vllm"
charly shell jupyter-ml -c "pixi run verify-unsloth"
charly shell jupyter-ml -c "pixi run verify-mcp"
charly shell jupyter-ml -c "pixi run verify-collaboration"
```

Inherits 3 deploy-scope `mcp:` checks from the candy (`ping`, `list-tools`,
`call notebook_list`); run `charly check live jupyter-ml --filter mcp`.

## Layout

- `charly.yml` — the `jupyter-ml:` candy entity plus its `skill:` entity.
- `pixi.toml` / `pixi.lock` — the Python environment (PyTorch, vLLM, unsloth,
  the data-science stack, spaCy).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-jupyter:jupyter-ml` — box properties, the ML stack, and
  verification.
- `/charly-jupyter:jupyter-mcp` — the CRDT MCP server extension and its tool
  catalog.
- `/charly-jupyter:jupyter-ml-notebook` — the same stack with fine-tuning
  notebooks.
- `/charly-jupyter:jupyter` — the lightweight CPU variant (no CUDA, multi-arch).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
