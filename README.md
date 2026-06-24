# Olympus

A self-hosted local AI stack that runs entirely on a single consumer GPU — no cloud dependencies, no API bills.

## What it is

Olympus wires together three open-source projects into one cohesive local assistant, plus a native multi-agent orchestrator on top. Everything runs through [Ollama](https://ollama.com) on a single GPU.

## Architecture

| Component | Role | Source |
|---|---|---|
| **Hermes** | Agent brain — memory, skills, tool execution | [`modules/hermes`](modules/hermes) ← [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) |
| **Odysseus** | Chat workspace UI | [`modules/odysseus`](modules/odysseus) ← [pewdiepie-archdaemon/odysseus](https://github.com/pewdiepie-archdaemon/odysseus) |
| **Jarvis** | Voice layer (hands-free) | [`modules/jarvis`](modules/jarvis) ← [isair/jarvis](https://github.com/isair/jarvis) |
| **Eris** | Multi-agent orchestrator — Doer/Challenger actor-critic pattern | [`eris/`](eris) — native to this repo |

Hermes, Odysseus, and Jarvis are git submodules, kept untouched upstream. Olympus only adds glue code — setup scripts, config, adapters — so integration work can eventually go back upstream as clean PRs instead of living forever as private forks. Eris lives natively here since it's original code, not a fork of anything.

## Hardware / Requirements

- Windows 11, native (no WSL)
- NVIDIA GPU, 16GB+ VRAM recommended (developed against an RTX 5060 Ti)
- [Ollama](https://ollama.com/download) installed and running
- A local model pulled — currently `qwen3:14b-q4_K_M`Any local model pulled via Ollama, sized to your VRAM — `qwen3:14b-q4_K_M` is just the current default on this machine
## Getting Started

```powershell
git clone --recurse-submodules https://github.com/lleoparden/olympus.git
cd olympus
```

Setup is still in progress — each submodule needs its own Python environment (see each one's own README under `modules/`). Current status, known gotchas, and session-by-session notes live in **[PROGRESS.md](PROGRESS.md)**.

## Status

🚧 Active development. Not yet a one-command install. Track progress in [PROGRESS.md](PROGRESS.md).

## Design Philosophy

- **Thin meta-repo** — only glue code lives here (setup, config, adapters, docs). Submodule internals are never touched directly.
- **Upstream-first** — integration work is written so it can be contributed back to Hermes/Odysseus/Jarvis as clean PRs, not maintained as silent forks.
- **Single-GPU reality** — design decisions (e.g. Eris running Doer→Challenger sequentially rather than in parallel) account for the fact that Ollama serializes inference on one consumer GPU regardless of how "parallel" the agent architecture looks on paper.
- **Model-agnostic** — nothing in Olympus hardcodes a model. Every component points at Ollama's local endpoint and can be repointed at whatever you've pulled — swapping models is a config change, not a code change.

## Credits

Built on top of:
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous Research (MIT)
- [Odysseus](https://github.com/pewdiepie-archdaemon/odysseus) (AGPL-3.0)
- [Jarvis](https://github.com/isair/jarvis) by isair (free for personal use)

## License

TBD for Olympus's own glue code. Each submodule keeps its own upstream license — check before redistributing any combined build.
