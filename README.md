# Nawirova — App

AI assistant untuk komputermu. Gratis selamanya. Tanpa internet.  
Your AI assistant. Free forever. Works offline.

---

## Apa ini? / What is this?

Nawirova adalah aplikasi desktop AI yang berjalan sepenuhnya di komputermu — tanpa koneksi internet, tanpa biaya per pesan, tanpa data yang dikirim ke server orang lain.

Nawirova is a desktop AI application that runs entirely on your computer — no internet connection after setup, no per-message cost, no data sent to external servers.

**Fitur / Features:**
- Chat — tanya apa saja / ask anything
- Translate — 30+ bahasa / 30+ languages
- Summarise — ringkas dokumen / summarise documents
- Write — tulis email, surat, laporan / write emails, letters, reports

## Persyaratan hardware / Hardware requirements

| Tier | RAM | Model |
|------|-----|-------|
| nano | 4–8 GB | Gemma 3 1B or Phi-4 mini 3.8B |
| base ⭐ | 8–16 GB | Qwen 2.5 7B |
| standard | 16–32 GB | Qwen 2.5 14B |
| extended | 32 GB+ | Gemma 3 12B or larger |

⭐ = target utama / primary target

**Platform:** Windows (primary), macOS, Linux  
**Minimum:** Windows 10 21H2+, 4 GB RAM, 5 GB disk free

## Unduh / Download

Lihat [Releases](https://github.com/nawirova/app/releases) untuk installer terbaru.  
See [Releases](https://github.com/nawirova/app/releases) for the latest installer.

| Platform | Format | Ukuran / Size |
|----------|--------|---------------|
| Windows | `.exe` (NSIS) | ~12 MB |
| macOS | `.dmg` | ~8 MB |
| Linux | `.AppImage` | ~14 MB |

Model AI diunduh secara otomatis saat pertama kali dibuka (~750 MB – 8 GB tergantung tier).  
The AI model downloads automatically on first launch (~750 MB – 8 GB depending on tier).

## Stack

| Layer | Technology |
|-------|-----------|
| Desktop shell | [Tauri 2](https://tauri.app) |
| Backend | Rust |
| Inference | [llama.cpp](https://github.com/ggerganov/llama.cpp) (sidecar) |
| Frontend | SvelteKit + Tailwind |
| Package manager | pnpm |

## Development

```bash
# Prerequisites: Rust, pnpm, Tauri CLI

git clone https://github.com/nawirova/app
cd app
pnpm install
pnpm tauri dev
```

## Kontribusi / Contributing

Lihat [CONTRIBUTING.md](CONTRIBUTING.md).  
Terjemahan baru, laporan bug, dan perbaikan sangat disambut.  
New translations, bug reports, and fixes are warmly welcomed.

## Lisensi / License

MIT OR Apache-2.0

---

*Nawirova — na-wee-RO-va*
