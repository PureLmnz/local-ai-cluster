
# Distributed Local AI Swarm Architecture

[![Network: Tailscale](https://img.shields.io/badge/Network-Tailscale%20Mesh-blue?style=flat-square&logo=tailscale)](https://tailscale.com)
[![Compute: NVIDIA RTX](https://img.shields.io/badge/Compute-NVIDIA%20RTX%204070-76B900?style=flat-square&logo=nvidia)](https://www.nvidia.com)
[![Protocol: OpenAI API](https://img.shields.io/badge/Protocol-OpenAI%20Compatible-black?style=flat-square)](https://github.com/openai/openai-python)
[![Orchestration: OpenCode](https://img.shields.io/badge/Orchestration-OpenCode-orange?style=flat-square)](https://github.com)

A highly optimized, multi-node distributed inference environment utilizing secure peer-to-peer mesh networking to route execution workloads between local edge devices and dedicated GPU server nodes. 

This project solves the hardware constraints of executing complex, multi-agent workflows locally by adopting an asynchronous **Tiered Router-Worker Pattern**. Lightweight orchestration runs locally on an M-series Mac, while heavy logical reasoning and code generation tasks are silently delegated to a headless Windows desktop GPU over a secure, zero-trust network.

---

## 📐 Architecture Overview

The system abstracts standard consumer hardware into a unified, parallel compute cluster operating entirely independent of paid cloud providers or external APIs.

```text
┌───────────────────────────────┐               ┌────────────────────────────────┐
│      LOCAL CLIENT NODE        │               │      REMOTE SPECIALIST NODE    │
│       (macOS / M5 Silicon)    │               │     (Windows / RTX 4070 GPU)   │
├───────────────────────────────┤               ├────────────────────────────────┤
│ • UI State & Orchestration    │   Tailscale   │ • Deep Reasoning & Code Gen    │
│ • Prompt Routing Logic        │ ──Mesh VPN──> │ • Headless LM Studio Server    │
│ • Fast Edge Models (Llama 3)  │  (Encrypted)  │ • Gemma 4 / Qwen 3.5 (Q4_K_M)  │
│ • OpenCode Runtime Harness    │               │ • Bound to 0.0.0.0:1234        │
└───────────────────────────────┘               └────────────────────────────────┘
```

### Core Design Principles
* **Zero-Latency Orchestration:** The client machine acts as an async router, handling context assembly and lightweight local models without hanging up local system threads.
* **Specialist Offloading:** Resource-intensive GGUF weights are kept persistently loaded in dedicated GPU VRAM on the remote node for maximum time-to-first-token performance.
* **Network Encapsulation:** All API payloads route through an encrypted Tailscale interface (`100.x.y.z`), eliminating the need to expose local ports to the public internet while bypassing standard NAT firewalls.

---

## 🧰 Hardware & Software Stack

### Hardware Nodes
* **Orchestrator Node:** Apple Silicon Mac (32GB Unified Memory)
* **Compute Node:** Dedicated Windows Machine housing an **NVIDIA GeForce RTX 4070 (12GB VRAM)**

### Model Execution & Routing
* **Server Backend:** [LM Studio](https://lmstudio.ai) configured as a headless inference server using the OpenAI REST API specification.
* **Primary Specialist Weights:** `Gemma 4` and `Qwen 3.5 (9B)` — Quantized to `Q4_K_M` GGUF format to maximize context lengths within the 12GB VRAM ceiling.
* **Local Runner:** Ollama executing highly compressed operational edge models.
* **Agent Harness:** OpenCode configured via customized `opencode.json` routing schemas and operational behavior profiles (`AGENTS.md`).

---

## ⚙️ Network & Server Configuration

To replicate this environment, both nodes must be authenticated to the same Tailscale tailnet. The remote inference backend must be explicitly bound to the network interface.

### 1. Binding the Inference Server
By default, local runners bind to loopback (`127.0.0.1`). To expose the server across the mesh network:
* **LM Studio:** Navigate to the Local Server Configuration and set the listener host interface to `0.0.0.0` on port `1234`.
* **Ollama (Alternative Backend):** Set the global system environment variable `OLLAMA_HOST=0.0.0.0` before service initialization.

### 2. Firewall Rules
Ensure inbound TCP traffic targeting your chosen API port is permitted through the host OS firewall specifically for the Tailscale adapter interface:
```powershell
# Windows Defender Firewall rule execution for mesh routing
New-NetFirewallRule -DisplayName "Tailscale Swarm Inbound" -Direction Inbound -LocalPort 1234 -Protocol TCP -Action Allow
```

---

## 🚀 Usage & Integration

### Asynchronous Parallel Swarm Execution
This pattern allows the local machine to query edge models while simultaneously executing heavy tasks on the remote GPU.

```python
import asyncio
from openai import AsyncOpenAI

# Initialize distinct network clients : (Using your own IP's and keys)
mac_node = AsyncOpenAI(base_url="http://localhost:11434/v1", api_key="local-edge") <-- Your own Key
windows_node = AsyncOpenAI(base_url="[http://100.](http://100.)x.y.z:1234/v1", api_key="remote-mesh") <-- Your own IP and Key

async def distributed_pipeline():
    # Task 1: Fast local routing/summarization
    local_task = mac_node.chat.completions.create(
        model="gemma4", <--Your model here
        messages=[{"role": "user", "content": "Classify this system event logic."}]
    )
    
    # Task 2: Heavy deterministic code generation offloaded to remote GPU
    remote_task = windows_node.chat.completions.create(
        model="gemma4",
        messages=[
            {"role": "system", "content": "You are a deterministic backend execution agent."},
            {"role": "user", "content": "Write a highly optimized execution pipeline logic class."}
        ],
        temperature=0.6
    )

    # Execute workloads concurrently
    local_res, remote_res = await asyncio.gather(local_task, remote_task)
    return local_res, remote_res

if __name__ == "__main__":
    asyncio.run(distributed_pipeline())
```

### OpenCode Harness Integration
To route native engineering agent workflows seamlessly through the remote compute node, configure the target client profile inside `opencode.json`:

```json
{
  "api_base": "[http://100.](http://100.)x.y.z:1234/v1", <-- Your own IP 
  "api_key": "mesh-node",  <-- Your own Key
  "model": "gemma4",  <-- Your model here
  "temperature": 0.6,
  "max_tokens": 8192,
  "global_settings": {
    "stream": true
  }
}
```

---

## 📊 Performance & Optimization Notes

* **VRAM Saturation:** Selecting `Q4_K_M` quantizations keeps active memory usage for a 9B parameter model strictly bounded to ~6GB. This leaves optimal buffer capacity for extended context windows (up to 32k tokens) during heavy source-code analysis tasks without spilling over into shared system memory.
* **Deterministic Configuration:** When targeting reasoning models via headless routing, global prompts enforce strict markdown block structure outputs while disabling unnecessary conversational fluff to optimize downstream script parsing.
* **Sampling Settings for Code Tasks:** For deterministic output logic over remote endpoints, temperature is clamped to `0.60` with `top_p` configured to `0.95`.

---

## 📜 License
Distributed under the MIT License. See `LICENSE` for more information.
