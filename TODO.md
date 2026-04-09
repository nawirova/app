# TODO

Versioned milestones. Tasks ordered by dependency within each section.  
Subtasks are atomic — one subtask = one commit or one PR.  
All PRs require passing CI before merge.

---

## Legend

- `[BLOCK]` — blocks downstream tasks
- `[SEC]` — security-relevant
- `[PLAT]` — platform-specific (W = Windows, M = macOS, L = Linux)
- `[DECISION]` — requires explicit decision before work starts

---

## Milestone v0.1.0 — First installable build

### 0. License & governance (prerequisite — do before any code)

- [x] Confirm license: MIT OR Apache-2.0 in `LICENSE-MIT` and `LICENSE-APACHE`
- [x] Add `SPDX-License-Identifier: MIT OR Apache-2.0` header policy to CONTRIBUTING.md
- [x] Add `SECURITY.md` — responsible disclosure contact, no bounty statement
- [x] Add `.github/CODEOWNERS` — assign as sole owner for v0.1.0
- [x] Add `.github/pull_request_template.md` — checklist: tests pass, i18n updated, platform tested

### 1. Repository scaffolding `[BLOCK]`

- [ ] `cargo init src-tauri` — Rust workspace, `tauri` + `sysinfo` dependencies
- [ ] `pnpm create svelte src` — SvelteKit inside Tauri project structure
- [ ] Add `pnpm-workspace.yaml`
- [ ] Add `.gitignore` — covers `node_modules/`, `target/`, `.env`, `dist/`, `*.gguf`
- [ ] Add `.editorconfig` — UTF-8, LF, 4-space indent Rust, 2-space TS/Svelte
- [ ] Add `Cargo.toml` — workspace manifest, Rust edition = 2021
- [ ] Add `package.json` — root scripts: `dev`, `build`, `tauri:dev`, `tauri:build`
- [ ] Verify `pnpm tauri dev` launches empty window on dev machine

### 2. CI/CD baseline `[BLOCK]`

- [ ] `.github/workflows/ci.yml` — trigger: push to `main` and all PRs
  - [ ] Job: `check` — `cargo check`, `cargo clippy -- -D warnings`
  - [ ] Job: `test` — `cargo test`, `pnpm test`
  - [ ] Job: `build` — `pnpm tauri build` on ubuntu-latest (smoke build)
  - [ ] Cache: `~/.cargo/registry`, `node_modules/` via actions/cache
- [ ] `.github/workflows/release.yml` — trigger: push tag `v*.*.*`
  - [ ] Build matrix: `windows-latest`, `macos-latest`, `ubuntu-latest`
  - [ ] Upload artifacts to GitHub Release draft
  - [ ] [SEC] Verify binaries built from tagged commit, not `main` tip
- [ ] Add `dependabot.yml` — weekly updates for `cargo` and `npm` ecosystems

### 3. Rust core — hardware `[BLOCK]`

- [ ] `src-tauri/src/hardware.rs`
  - [ ] `total_ram_gb() -> f64` via `sysinfo::System::total_memory()`
  - [ ] `detect_tier(ram_gb: f64) -> Tier` — enum: Nano / Base / Standard / Extended
  - [ ] [PLAT-M] Detect Apple Silicon via `sysctl hw.optional.arm64` — bump tier one level
  - [ ] Unit test: `test_tier_detection` — 4, 8, 16, 32, 64 GB inputs
  - [ ] Unit test: `test_apple_silicon_tier_bump`
- [ ] `src-tauri/src/models.rs`
  - [ ] Struct `ModelEntry` — deserialises from `registry.toml`
  - [ ] `load_registry() -> Vec<ModelEntry>` — reads bundled TOML fallback
  - [ ] `select_model(tier: Tier) -> ModelEntry`
  - [ ] Unit test: `test_registry_loads` — all 5 models present
  - [ ] Unit test: `test_model_selection_per_tier`

### 4. Rust core — download manager

- [ ] `src-tauri/src/download.rs`
  - [ ] `model_path(model_id: &str) -> PathBuf` — platform-correct AppData path
    - [ ] [PLAT-W] `%APPDATA%\Nawirova\models\`
    - [ ] [PLAT-M] `~/Library/Application Support/Nawirova/models/`
    - [ ] [PLAT-L] `~/.local/share/nawirova/models/`
  - [ ] `is_model_present(path: &Path, expected_sha256: &str) -> bool`
    - [ ] [SEC] SHA-256 verify via `sha2` crate — never skip on startup
  - [ ] `async download_model(url, dest, tx: Sender<DownloadProgress>)`
    - [ ] Send `Range: bytes={offset}-` header if partial file exists
    - [ ] Emit `DownloadProgress { bytes_done, bytes_total, pct }` via channel
    - [ ] Atomic write: download to `.part`, rename on completion + verify
  - [ ] [SEC] Reject downloads from any URL not matching `huggingface.co` or `hf.co`
  - [ ] Unit test: `test_resume_logic` — mock partial file, assert Range header sent
  - [ ] Unit test: `test_sha256_reject` — corrupt file, assert verification fails

### 5. Rust core — inference (llama.cpp sidecar)

- [ ] Source pre-built `llama-server` binaries
  - [ ] [PLAT-W] `llama-server-windows-x64.exe` from llama.cpp releases
  - [ ] [PLAT-M] `llama-server-macos-arm64` and `llama-server-macos-x64`
  - [ ] [PLAT-L] `llama-server-linux-x64`
  - [ ] [SEC] Verify SHA-256 of each binary against llama.cpp release checksums
  - [ ] Place in `src-tauri/binaries/` with Tauri target-triple naming
- [ ] Register sidecar under `externalBin` in `tauri.conf.json`
- [ ] `src-tauri/src/inference.rs`
  - [ ] `start_server(model_path, port: u16) -> Result<Child>`
    - [ ] Bind to `127.0.0.1:{port}` — never `0.0.0.0`
    - [ ] Port: 8765 (avoids Ollama collision on 11434)
  - [ ] `wait_ready(port, timeout_ms) -> Result<()>` — poll `/health`
  - [ ] `stop_server(child: &mut Child)` — SIGTERM, wait 3 s, SIGKILL
  - [ ] [SEC] Assert sidecar process is owned by current user before accepting responses
  - [ ] Integration test: `test_server_starts_and_responds`

### 6. Rust core — IPC commands

- [ ] `src-tauri/src/commands.rs`
  - [ ] `get_hardware_info() -> HardwareInfo`
  - [ ] `async chat(messages, options) -> Result<String>` — stream via Tauri events
  - [ ] `async translate(text, from, to) -> Result<String>`
  - [ ] `async summarise(text) -> Result<String>`
  - [ ] `async write(template, to, topic, tone) -> Result<String>`
  - [ ] `get_download_progress() -> Option<DownloadProgress>`
  - [ ] `async cancel_download() -> Result<()>`
- [ ] Register all commands in `main.rs` `invoke_handler`

### 7. Rust core — context persistence

- [ ] `src-tauri/src/context.rs`
  - [ ] Struct `Turn { role, content, timestamp_ms }`
  - [ ] `history_path() -> PathBuf` — same AppData base as model path
  - [ ] `load_history(max_turns: usize) -> Vec<Turn>` — NDJSON, last N lines
  - [ ] `append_turn(turn: &Turn) -> Result<()>` — append single JSON line
  - [ ] `clear_history() -> Result<()>`
  - [ ] Unit test: `test_history_round_trip` — write 5, reload, assert order

### 8. SvelteKit frontend — scaffold

- [ ] Configure Tailwind CSS
- [ ] `src/lib/stores/conversation.ts` — writable store for chat turns
- [ ] `src/lib/stores/app.ts` — tier, download state, ready state
- [ ] `src/lib/i18n.ts` — locale loader, reads from `localStorage`
- [ ] `src/lib/i18n/en.json` — all UI strings `[BLOCK for all routes]`
- [ ] `src/lib/i18n/id.json` — Bahasa Indonesia (complete before v0.1.0)
- [ ] `src/lib/tauri.ts` — typed wrappers around `@tauri-apps/api/core` invoke

### 9. SvelteKit frontend — onboarding flow

- [ ] `src/routes/onboarding/+page.svelte` — step controller (4 steps)
- [ ] `Welcome.svelte` — logo, tagline, 4 feature icons, "Get started"
- [ ] `Setup.svelte` — detected RAM, tier + model name + disk size, GPU note
- [ ] `Download.svelte`
  - [ ] Progress bar polling `get_download_progress` every 500 ms
  - [ ] Estimated time remaining label
  - [ ] "Continue in background" — navigates to app, download continues
  - [ ] Error state with retry button
- [ ] `Ready.svelte` — CSS checkmark, model pill, "Start chatting"

### 10. SvelteKit frontend — main layout

- [ ] `src/routes/+layout.svelte` — sidebar + content
- [ ] `Sidebar.svelte`
  - [ ] Nav: Chat, Translate, Summarise, Write
  - [ ] Active state
  - [ ] Language selector (EN / ID)
  - [ ] Settings button (disabled in v0.1.0)

### 11. SvelteKit frontend — Chat route

- [ ] `src/routes/+page.svelte`
- [ ] `MessageList.svelte` — renders turns, auto-scrolls to bottom
- [ ] `MessageBubble.svelte` — user / assistant visual distinction
- [ ] `TypingIndicator.svelte` — 3-dot CSS animation
- [ ] `SuggestionGrid.svelte` — 4 cards shown on empty state
- [ ] `ChatInput.svelte` — textarea, Enter sends, Shift+Enter newline, disabled while streaming
- [ ] Connect `chat` command, listen `chat-chunk` events, append to store

### 12. SvelteKit frontend — Translate route

- [ ] `src/routes/translate/+page.svelte`
- [ ] `LangSelector.svelte` — 30-language dropdown (list in ARCHITECTURE.md)
- [ ] `SwapButton.svelte` — swap from/to
- [ ] Input textarea / output display two-panel layout
- [ ] Connect `translate` command

### 13. SvelteKit frontend — Summarise route

- [ ] `src/routes/summarise/+page.svelte`
- [ ] `DropZone.svelte` — accept `.txt`, `.md` in v0.1.0; drag-and-drop + click
- [ ] File size guard: reject > 500 KB, show error message
- [ ] Paste textarea alternative
- [ ] Connect `summarise` command

### 14. SvelteKit frontend — Write route

- [ ] `src/routes/write/+page.svelte`
- [ ] `TemplateSelector.svelte` — Email / Letter / Report
- [ ] Two-field form: "To" + "What to say"
- [ ] Tone selector: Professional / Friendly / Formal
- [ ] Connect `write` command
- [ ] "Copy text" button

### 15. Installer — Windows `[PLAT-W]`

- [ ] Configure Tauri NSIS bundler in `tauri.conf.json`
- [ ] Test install on Windows 10 21H2 clean VM
- [ ] Test install on Windows 11 clean VM
- [ ] [SEC] Embed WebView2 bootstrapper for offline installs
- [ ] [SEC] Obtain EV code-signing certificate
- [ ] Sign installer binary before uploading release artifact
- [ ] Smoke test: install → launch → onboarding → send one chat message

### 16. Installer — macOS `[PLAT-M]`

- [ ] Configure Tauri `.dmg` bundler
- [ ] [SEC] Notarisation via `notarytool` in `release.yml`
- [ ] Test on macOS 13 Intel + macOS 14 Apple Silicon
- [ ] Smoke test: drag to Applications, launch, onboarding completes

### 17. Installer — Linux `[PLAT-L]`

- [ ] Configure Tauri AppImage bundler
- [ ] Detect missing `libwebkit2gtk-4.1-0` on launch — show install prompt
- [ ] Test on Ubuntu 22.04 and Ubuntu 24.04
- [ ] Document `chmod +x` requirement in README
- [ ] Smoke test: AppImage → onboarding completes

### 18. v0.1.0 quality gates (all must pass before tagging)

- [ ] [SEC] `cargo audit` — zero high/critical advisories
- [ ] [SEC] `pnpm audit` — zero high/critical advisories
- [ ] Download resume: start 4.5 GB download, kill mid-way, restart, verify continuation
- [ ] 8 GB RAM smoke test: nano tier, Phi-4 mini, all 4 features — no OOM
- [ ] Cold start: model loaded → chat input ready < 3 s (5 runs, median)
- [ ] Context persistence: 3 messages → close → reopen → history present
- [ ] Offline: disable network after model download, all 4 features work
- [ ] i18n completeness: EN and ID — zero untranslated keys

---

## Milestone v0.2.0 — Post-launch hardening

### Features

- [ ] Settings panel: language, tier override, context length, clear history
- [ ] Summarise: PDF support via platform WebView PDF API or `pdf-extract` crate
- [ ] Summarise: DOCX support via `docx` crate
- [ ] Copy-to-clipboard on all output panels (Tauri Clipboard plugin)
- [ ] Local feedback: thumbs up/down stored in SQLite, never transmitted
- [ ] Hindi translation `hi.json` — complete
- [ ] Brazilian Portuguese translation `pt_br.json` — complete
- [ ] Auto-update: in-app notification + link to release page (opt-in only)

### DevSecOps

- [ ] Add `cargo-deny` to CI — licence allowlist, duplicate dependency check
- [ ] Add `cargo-cyclonedx` — SBOM generation on release
- [ ] Attach SPDX SBOM to GitHub Release assets
- [ ] Integration test suite: each feature sends real prompt to llama-server in CI

---

## Milestone v0.3.0 — Stretch

- [ ] Swahili translation `sw.json`
- [ ] Voice input: Whisper.cpp sidecar, on-device transcription, mic permission
- [ ] Image input for Summarise: Gemma 3 multimodal on base+ tier
- [ ] Multiple conversation tabs (max 5, persisted separately)
- [ ] Export conversation: `.txt` and `.pdf` via Tauri dialog + printToPdf

---

## Open decisions `[DECISION]` (resolve before implementing dependent tasks)

- [ ] **Short form** — `Nawi` nickname in UI, or full name `Nawirova` only?
  → Affects: sidebar title, onboarding heading, window title
- [ ] **Default language** — system locale detection, or always show language picker?
  → Affects: `src/lib/i18n.ts` init logic
- [ ] **Model update cadence** — how does user learn a better model is available?
  → Options: (a) check `nawirova/models` registry on launch, (b) manual "Check updates" only, (c) never
- [ ] **Linux `.deb`** — ship alongside AppImage for Ubuntu users?
  → Low effort via Tauri bundler. +1 artifact + +1 install test per release.
- [ ] **Context window defaults** — 8192 tokens nano, 16384 base; user-adjustable in v0.2.0?
