# Contributing to Nawirova

Terima kasih atas minat Anda untuk berkontribusi!  
Thank you for your interest in contributing!

---

## License Header Policy

All source files must include the following SPDX license identifier at the top:

```
// SPDX-License-Identifier: MIT OR Apache-2.0
```

For Rust files:
```rust
// SPDX-License-Identifier: MIT OR Apache-2.0
```

For TypeScript/Svelte files:
```typescript
// SPDX-License-Identifier: MIT OR Apache-2.0
```

---

## How to Contribute

### Reporting Bugs

1. Check if the issue already exists in [Issues](https://github.com/nawirova/app/issues)
2. Create a new issue with:
   - Clear title and description
   - Steps to reproduce
   - Expected vs actual behavior
   - Platform (Windows/macOS/Linux) and version
   - Screenshots if applicable

### Suggesting Features

1. Open a discussion in [Discussions](https://github.com/nawirova/app/discussions) first
2. If there's consensus, create an issue with the `enhancement` label

### Pull Requests

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Add/update tests as needed
5. Ensure all tests pass: `cargo test` and `pnpm test`
6. Update translations if UI strings changed
7. Commit with clear messages
8. Push and open a PR against `main`

---

## Development Setup

See [README.md](README.md#development) for prerequisites and setup instructions.

---

## Code Style

### Rust
- Follow the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- Run `cargo fmt` and `cargo clippy` before committing
- 4-space indentation

### TypeScript/Svelte
- Follow the project's existing patterns
- 2-space indentation
- Run `pnpm check` before committing

---

## Translations

New translations are warmly welcomed! See `src/i18n/` for existing translation files.

---

## Questions?

Open a discussion in [GitHub Discussions](https://github.com/nawirova/app/discussions).
