# Ground Control (GCTRL) — Deployment

**Stateless knowledge-graph and memory middleware for AI and agents. On-prem, local inference, GDPR-ready.**

This repository holds the deployment and management scripts for [Ground Control](https://gctrl.tech). The canonical way to install is the one-line installer below; the scripts here cover updates, rollbacks, and removal.

- Website: https://gctrl.tech
- Documentation: https://gctrl.tech/docs
- Use cases: https://gctrl.tech/use-cases

---

## What is GCTRL

GCTRL is a stateless middleware layer that sits on top of your own storage. It ingests content at scale, builds a knowledge graph, and organises layered memory — hot dossiers, warm chunks, cold graph, and a curated Wiki — then serves all of it to your agents over MCP, with full classification and clearance-based access control plus audit.

Everything runs on-prem. Inference stays local. Your data never leaves your infrastructure.

The platform bundles its storage (Neo4j for the graph, Qdrant for vectors, Postgres, and Redis), but every backing store is swappable — point GCTRL at your existing Neo4j, Qdrant, or inference endpoint and it will use them instead.

---

## Quick start

On macOS or Linux, install with a single command:

```bash
curl -fsSL https://gctrl.tech/install | bash
```

The installer:

1. Detects existing **Neo4j**, **Qdrant**, and **Ollama** services and reuses them — otherwise it deploys the bundled containers.
2. Detects your **GPU**.
3. Shows an **interactive model picker** so you can choose the local model to pull.
4. Brings up the **full stack** via Docker Compose.
5. Deploys the **FUSE** resolution engine automatically.

To skip the model picker (for scripted or repeatable installs), set the model up front:

```bash
GCTRL_MODEL=qwen2.5:7b curl -fsSL https://gctrl.tech/install | bash
```

When it finishes, open **http://localhost:3001**, create your admin account, and activate your license.

---

## Get the full power: run Ollama natively

This is the single most important step for real-world performance.

The bundled Ollama runs **inside Docker, which is CPU-only**. Docker cannot reach the host GPU — not Apple Metal on macOS, and not NVIDIA on Windows or Linux. The bundled container is fine to get started, but it is slow for serious work.

For full performance you must install Ollama **natively on the host** and point GCTRL at it. One switch in the dashboard repoints **both** the RAG/agent path **and** KEX extraction and embeddings to the native, GPU-accelerated Ollama.

### Step 1 — Install native Ollama for your OS

**macOS (Apple Silicon)**

Install the native Ollama app from https://ollama.com/download, or via Homebrew:

```bash
brew install ollama
```

It uses the Metal GPU and unified memory. Then pull a model:

```bash
ollama pull qwen2.5:7b
```

On large-RAM machines you can pull `qwen2.5:14b` or `qwen2.5:32b`.

**Windows**

Install Ollama for Windows (native, NVIDIA-GPU accelerated) from https://ollama.com/download. Then in PowerShell:

```powershell
ollama pull qwen2.5:7b
```

See [WINDOWS.md](WINDOWS.md) for the complete Windows walkthrough.

**Linux**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

This installs Ollama with NVIDIA GPU acceleration (drivers required). Then:

```bash
ollama pull qwen2.5:7b
```

### Step 2 — Point GCTRL at native Ollama

In the dashboard, go to **Settings → Infrastructure** and set the Ollama base URL to:

```
http://host.docker.internal:11434
```

Save. GCTRL now routes all inference — RAG, agents, KEX extraction, and embeddings — to your native, GPU-accelerated Ollama.

Without this step, GCTRL uses the bundled CPU-only Ollama: fine to start, slow for real work.

---

## Requirements

| Requirement | Notes |
|-------------|-------|
| **Docker** + **docker compose** plugin | The stack runs as containers. The modern compose plugin (`docker compose`) is required, not legacy `docker-compose`. |
| **curl** | Fetches the installer. |
| **openssl** | Generates local secrets and keys. |
| **16 GB+ RAM** | Minimum. Larger models need more — budget accordingly. |
| **GPU (recommended)** | Apple Metal (macOS), NVIDIA (Windows/Linux) — used via native Ollama (see above). |

---

## Activate your license

After the stack is up:

1. Open **http://localhost:3001**.
2. Create your admin account.
3. Enter your license key in the format:

   ```
   GCTRL-XXXX-XXXX-XXXX-XXXX-XXXX
   ```

The license agent runs locally on port `7070` and validates your key.

---

## Platform guides

### macOS / Linux

Use the one-line installer:

```bash
curl -fsSL https://gctrl.tech/install | bash
```

### Windows

Windows runs GCTRL through Docker Desktop with the WSL 2 backend. See **[WINDOWS.md](WINDOWS.md)** for the full step-by-step guide, or the docs version at https://gctrl.tech/docs/windows.

---

## Manage

This repository provides management scripts for an installed deployment:

| Script | Purpose |
|--------|---------|
| `update.sh` | Pull the latest images and apply updates with minimal downtime. |
| `uninstall.sh` | Remove GCTRL containers and (optionally) volumes. |
| `rollback.sh` | Revert to the previous known-good release. |
| `compose-template.yml` | Reference Compose definition for the full stack. |

> `compose-template.yml` is a reference. The canonical install path is the one-line installer above, which generates and manages your Compose configuration for you.

Run a script from the repository root, for example:

```bash
./update.sh
```

---

## Ports

| Service | Port |
|---------|------|
| Dashboard | `3001` |
| API | `4000` |
| KEX (extraction) | `4010` |
| FUSE (fusion / entity resolution) | `4020` |
| License agent | `7070` |

---

## Architecture

GCTRL is **stateless middleware** over swappable storage. The bundled stack ships Neo4j (graph), Qdrant (vectors), Postgres, and Redis, but each can be replaced with your own.

The flow:

```
Sources → KEX → FUSE → Layered memory → Agents (MCP)
```

- **KEX** extracts structured knowledge from your sources and builds the graph.
- **FUSE** resolves and merges entities into a single, deduplicated knowledge graph.
- **Layered memory** organises everything into hot dossiers, warm chunks, the cold graph, and a curated Wiki.
- **Agents** consume it over MCP, gated by classification and clearance, with full audit.

---

## Learn more

- Website — https://gctrl.tech
- Documentation — https://gctrl.tech/docs
- Windows install guide — https://gctrl.tech/docs/windows
- Infrastructure and Ollama — https://gctrl.tech/docs/infrastructure
- Use cases — https://gctrl.tech/use-cases
