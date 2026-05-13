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
