# TODO

Ordered by dependency. Items within a section are roughly parallel.

---

## v0.1 — First working build

### Rust core
- [ ] `hardware.rs` — RAM detection via `sysinfo` crate, tier selection logic
- [ ] `download.rs` — resumable download, `Range` header, SHA-256 verify
- [ ] `inference.rs` — llama.cpp sidecar spawn, health check, graceful shutdown
- [ ] `commands.rs` — Tauri IPC: `chat`, `get_hardware_info`, `get_download_progress`, `cancel_download`
- [ ] `context.rs` — conversation history, NDJSON read/write

### llama.cpp sidecar
- [ ] Download pre-built `llama-server` binaries for Windows x64, macOS arm64, Linux x64
- [ ] Bundle via Tauri `externalBin` in `tauri.conf.json`
- [ ] Verify sidecar starts correctly on each platform

### SvelteKit frontend
- [ ] Scaffold with `pnpm create svelte` inside Tauri project
- [ ] Chat route — message input, streaming response display, typing indicator
- [ ] Translate route — language pair selector (30 languages), input/output panels
- [ ] Summarise route — file drop zone + paste textarea, summary output
- [ ] Write route — template selector (Email/Letter/Report), two-field form, output
- [ ] Sidebar navigation, active state
- [ ] i18n scaffold — `en.json`, `id.json` string files

### Onboarding
- [ ] Hardware detection screen (shows detected RAM + selected tier)
- [ ] Model download screen (progress bar, resumable, "Continue in background")
- [ ] Ready screen (checkmark, "Start chatting" CTA)

### Installer
- [ ] Windows NSIS — test on Windows 10 21H2 and Windows 11
- [ ] macOS `.dmg` — notarisation flow
- [ ] Linux `.AppImage` — WebKitGTK dependency check + install prompt
- [ ] Code-signing setup (Windows EV cert — eliminates SmartScreen warning)

---

## v0.1 — Quality bar before public release

- [ ] Classifier accuracy: heuristic routing ≥ 85% on test suite
- [ ] Download resume: interrupt mid-download, restart, verify correct continuation
- [ ] 8 GB RAM smoke test: full flow on nano tier, no OOM
- [ ] Cold start time: < 3 seconds from launch to chat input ready (model already loaded)
- [ ] Context persistence: close and reopen app, last conversation restored
- [ ] Offline verification: pull ethernet, confirm all features work
- [ ] i18n: EN and ID strings complete before release

---

## v0.2 — Post-launch

- [ ] Settings panel (language, tier override, context length, clear history)
- [ ] File input for Summarise (PDF, DOCX parsing)
- [ ] Copy-to-clipboard button on all outputs
- [ ] "Rate this response" for local quality tracking (no data leaves device)
- [ ] Hindi (`hi.json`) and Brazilian Portuguese (`pt_br.json`) translations
- [ ] Auto-update check (opt-in, manual trigger)
- [ ] Swahili (`sw.json`) translation

---

## v0.3 — Stretch

- [ ] Voice input (Whisper.cpp sidecar, on-device transcription)
- [ ] Image input for Summarise (Gemma 3 multimodal on base+ tier)
- [ ] FIM (fill-in-the-middle) mode for developer users
- [ ] Multiple conversation tabs
- [ ] Export conversation as PDF or plain text

---

## Decisions needed (not blocked, but resolve before v0.1)

- [ ] Short form: does `Nawirova` get a nickname (`Nawi`) in the UI, or full name only?
- [ ] Default language: detect from system locale at first launch, or always prompt?
- [ ] Model update cadence: how does the app know when a better model is available for its tier?
- [ ] Linux `.deb` package: worth providing alongside AppImage for Ubuntu users who dislike AppImage?
