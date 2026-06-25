# Olympus — Progress Log

> Self-hosted local AI stack. Single GPU (RTX 5060 Ti, 16GB), Windows 11 native, Ollama-served Qwen3-14B Q4_K_M.
> Update the **Session Log** at the bottom each time we work on this. Everything above it is a current-state snapshot — edit in place as things change, don't duplicate entries.

## Architecture

| Component | Role | Source |
|---|---|---|
| **Hermes** | Agent brain — memory, skills, tool execution | `modules/hermes` ← NousResearch/hermes-agent |
| **Odysseus** | Chat workspace UI | `modules/odysseus` ← pewdiepie-archdaemon/odysseus |
| **Jarvis** | Voice layer | `modules/jarvis` ← isair/jarvis |
| **Eris** | Multi-agent orchestrator (Doer/Challenger actor-critic) | `eris/` — native to Olympus, not a submodule |

All three submodules stay untouched upstream; integration work lives in Olympus glue code only, for clean PR-back later.

## Environment

- Windows 11, native (no WSL)
- RTX 5060 Ti, 16GB VRAM — Blackwell/sm_120, CUDA compute 12.0
- Ollama (native Windows, auto-detects GPU, zero config)
- GitHub: `lleoparden`

## Open Design Decisions

- **Eris pattern:** Doer/Challenger replacing the council model. Challenger stays strictly advisory (critique only, no rewriting) to preserve clear ownership of final output.
  - Still open: single critique pass vs. iterative loop with a round budget cap.
  - Still open: does Doer/Challenger fully replace the council model, or become a selectable mode within Eris by task type?

## Known Gotchas

- **PyTorch + Blackwell (sm_120):** stable PyTorch wheels still don't officially support the 5060 Ti. Only matters if a component pulls in raw PyTorch directly — not Ollama, not llama.cpp/GGML/CTranslate2-backed tools. Watch for this if Hermes or Jarvis grow embedding/STT features that need it.
- **Odysseus `chromadb-client` conflict:** their `requirements.txt` installs the HTTP-only `chromadb-client` by default, meant for the Docker Compose setup where ChromaDB runs as its own container. Native installs need the full `chromadb` package instead, or memory silently degrades. Fix: `pip uninstall chromadb-client -y && pip install --force-reinstall chromadb`.
- **Hermes `uv.exe` AV false positive:** some antivirus (Defender, Bitdefender) flag the bundled Rust `uv` binary. Known false positive — verification steps are in Hermes's README if it comes up.
- **winget installs need a fresh terminal:** PATH changes don't apply to already-open PowerShell windows. Close/reopen before using a newly installed CLI tool.
- **Jarvis dev is macOS-first:** per their own README, Windows/Linux support may lag behind.

## Roadmap

- [ ] Finish per-component native env setup (Hermes/Odysseus/Jarvis) — paused mid-session, resume next
- [ ] Wire Ollama endpoint into all three (Odysseus Settings, Hermes `hermes model`, Jarvis setup wizard)
- [ ] Design + build Eris (Doer/Challenger orchestrator)
- [ ] **Docker Compose for the full stack** (Ollama + Hermes + Odysseus + Jarvis + Eris) — future work, not started.
  - Odysseus and Hermes already ship their own Dockerfiles/compose configs upstream — the Olympus-level compose should probably *extend/reference* those rather than rewrite them, in keeping with the thin-meta-repo philosophy.
  - Open question: Jarvis is a desktop app needing mic input, global hotkeys, and a system tray icon — none of that plays nicely inside a container, especially on Windows. May end up being the one component that stays native instead of containerized. Decide when we get there.
- [ ] Upstream PRs once integration code stabilizes (e.g. Jarvis LLM endpoint flag mentioned earlier)

---

## Session Log

### 2026-06-24 — Session 1
- Created `olympus` GitHub repo (meta-repo), pushed initial skeleton (`modules/`, `eris/`, `scripts/`, `config/`, `docs/`, `.gitignore`)
- Phase 1 machine prep: Git, GitHub CLI, Node LTS, Python 3.12, Ollama — all via winget. Confirmed GPU detected by Ollama (CUDA, compute 12.0, 15.9GB VRAM).
- Renamed orchestrator Hydra → **Eris** (staying within Greek/Olympus naming; fits the actor-critic "discord" framing too)
- Added all three submodules: `modules/hermes`, `modules/odysseus`, `modules/jarvis`
- Started per-component env setup — hit two snags, paused here:
  - `uv` not found after winget install (stale PATH in an already-open terminal)
  - Odysseus: only Python 3.12 installed, not 3.11 (fine — Odysseus supports 3.11+)
  - Also queued: swap `chromadb-client` → `chromadb` in Odysseus's native install
- **Next session starts here:** redo Hermes/Odysseus env setup with the fixes above, then Jarvis (`run_windows.ps1`)

## Session — 2026-06-26

**Goal:** Resume per-component Python environment setup (paused from previous session due to uv PATH staleness + Python version mismatches).

**Done:**
- Created `.venv` (Python 3.12.10) for all four components: Hermes, Odysseus, Jarvis, Eris
- Hermes: `uv pip install -e .` — clean install, 64 packages, no issues
- Odysseus: editable install initially failed (`ModuleNotFoundError: httpx`) because `core/__init__.py` imports happen at setup-time, before build isolation has the package's deps. Fixed via:
  - `uv pip install --python .venv\Scripts\python.exe httpx setuptools wheel`
  - `uv pip install --python .venv\Scripts\python.exe -e . --no-build-isolation`
  - `uv venv` does not pre-seed setuptools/wheel — needed explicitly for `--no-build-isolation` builds
- Jarvis: not a packaged project (no `pyproject.toml`/`setup.py`), installs via `requirements.txt`. `webrtcvad==2.0.10` fails to build on Windows (needs MSVC C++ Build Tools, no prebuilt wheel). Fixed without touching the submodule's tracked `requirements.txt`:
  - Installed `webrtcvad-wheels==2.0.14` (prebuilt-wheel fork, same `webrtcvad` import namespace)
  - Filtered the `webrtcvad==2.0.10` line out via `findstr /v` into a local copy (`jarvis-requirements-filtered.txt`, outside the submodule) and installed from that
- Eris: confirmed empty except `.venv` — no code, no `pyproject.toml` yet. Not a blocker; Doer/Challenger orchestrator hasn't been scaffolded yet.

**Gotchas logged for next time:**
- `uv venv` does NOT include setuptools/wheel by default — needed for any `--no-build-isolation` install
- macOS-first packages (like Jarvis's `webrtcvad`) may need a Windows-wheel fork substitute rather than a straight pin fix
- Use `--python .venv\Scripts\python.exe` explicitly with `uv pip install` instead of activating — PowerShell execution policy blocks `.venv\Scripts\activate.ps1` by default on this machine

**Next session:**
- Scaffold Eris: `pyproject.toml`, package structure, start Doer/Challenger orchestration design
