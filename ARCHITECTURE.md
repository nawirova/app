# Architecture

## Overview

Nawirova is a Tauri 2 desktop application. All AI inference runs locally via a
bundled llama.cpp sidecar binary. No network calls occur after the initial model
download. The app has no backend server, no telemetry, and no cloud dependency.

```
┌─────────────────────────────────────────────┐
│  Nawirova Desktop App                       │
│                                             │
│  ┌──────────────┐    IPC    ┌─────────────┐ │
│  │  SvelteKit   │ ◄──────► │  Rust core  │ │
│  │  (WebView2 / │           │  (Tauri 2)  │ │
│  │   WebKit)    │           └──────┬──────┘ │
│  └──────────────┘                  │        │
│                              stdio / HTTP   │
│                                    │        │
│                           ┌────────▼──────┐ │
│                           │  llama.cpp    │ │
│                           │  (sidecar)    │ │
│                           └───────────────┘ │
└─────────────────────────────────────────────┘
         │                         │
    User's disk              User's disk
    (model files)            (chat history)
```

## Components

### Rust core (`src-tauri/src/`)

| Module | Responsibility |
|--------|---------------|
| `main.rs` | Tauri app init, window setup |
| `hardware.rs` | RAM detection, tier selection |
| `inference.rs` | llama.cpp sidecar lifecycle (start, stop, health) |
| `download.rs` | Resumable model download via `reqwest` + `Range` header |
| `context.rs` | Conversation history, serialisation to disk |
| `commands.rs` | Tauri IPC commands exposed to frontend |

### llama.cpp sidecar

A pre-built `llama-server` binary is bundled per platform inside
`src-tauri/binaries/`. Tauri's sidecar mechanism launches it on app start and
kills it on app exit. It binds to `127.0.0.1:8765` (not 11434 — avoids
collision with Ollama if installed).

The sidecar exposes an OpenAI-compatible `/v1/chat/completions` endpoint.
The Rust core calls this endpoint; the frontend never calls it directly.

### SvelteKit frontend (`src/`)

| Route | Feature |
|-------|---------|
| `/` | Chat |
| `/translate` | Translation (language pair selector) |
| `/summarise` | Document summarisation (file drop + paste) |
| `/write` | Structured document generation (template + form) |

All routes share a single Svelte store for conversation context. The store is
serialised by `context.rs` between sessions.

### Model registry

Model metadata is not hardcoded in the app. At first launch, the app reads
`models/registry.toml` (bundled as a fallback) and optionally fetches an
updated version from `github.com/nawirova/models`. After first launch, the
app works fully offline — no registry update attempts are made unless the user
explicitly triggers "Check for updates" in Settings.

## Hardware tier detection

```
startup
   │
   ▼
read total system RAM (psutil / sysinfo)
   │
   ├─ < 8 GB  → tier: nano   → model: Gemma 3 1B   (750 MB)
   ├─ 8–16 GB → tier: base   → model: Qwen 2.5 7B  (4.5 GB)  ← primary
   ├─ 16–32 GB→ tier: standard→ model: Qwen 2.5 14B (8.9 GB)
   └─ > 32 GB → tier: extended→ model: Gemma 3 12B  (7.3 GB)
   │
   ▼
show tier + model to user (with manual override option)
   │
   ▼
download model if not present (resumable, progress bar)
   │
   ▼
launch llama.cpp sidecar
   │
   ▼
app ready
```

## Model download

Downloads target `%APPDATA%\Nawirova\models\` on Windows,
`~/Library/Application Support/Nawirova/models/` on macOS,
`~/.local/share/nawirova/models/` on Linux.

Resume logic: before each download chunk, the app checks local file size and
sends `Range: bytes={offset}-` in the HTTP request. A corrupted partial
download is detected via SHA-256 checksum against the registry manifest.

## Context persistence

Chat history is stored as newline-delimited JSON in:
- Windows: `%APPDATA%\Nawirova\history.ndjson`
- macOS: `~/Library/Application Support/Nawirova/history.ndjson`
- Linux: `~/.local/share/nawirova/history.ndjson`

Maximum retained turns: 50 (configurable in Settings). The llama.cpp sidecar
receives only the last N turns on each request, not the full history.

## IPC contract

All frontend ↔ Rust communication goes through Tauri commands:

```typescript
// Frontend calls
invoke('chat', { messages, options })
invoke('translate', { text, from_lang, to_lang })
invoke('summarise', { text })
invoke('write', { template, to, topic, tone })
invoke('get_hardware_info')       // → { ram_gb, tier, model }
invoke('get_download_progress')   // → { pct, bytes_done, bytes_total }
invoke('cancel_download')
```

No frontend code calls llama.cpp directly. All inference is mediated by Rust.

## Security

- llama.cpp sidecar binds to `127.0.0.1` only — not accessible from the network
- No Tauri capabilities grant network access to the frontend
- No external URLs are opened without explicit user action
- Model files are verified by SHA-256 before the sidecar loads them
- No telemetry, no crash reporting, no analytics of any kind

## Platform notes

| Platform | WebView | Code-signing | Auto-update |
|----------|---------|-------------|-------------|
| Windows | WebView2 (Edge, pre-installed Win10+) | EV cert required to avoid SmartScreen | Tauri updater (NSIS/MSI) |
| macOS | WebKit (system) | Notarisation required | Tauri updater (.app bundle) |
| Linux | WebKitGTK (may need install) | Not required | Manual re-download or AppImageUpdate |
