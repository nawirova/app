# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 0.1.x   | :white_check_mark: |
| < 0.1.0 | :x:                |

---

## Reporting a Vulnerability

If you discover a security vulnerability in Nawirova, please report it responsibly.

**Do NOT** open a public issue for security vulnerabilities.

### Contact

Email: **security@nawirova.app**

Please include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)
- Your disclosure timeline preference

### Response Timeline

- **Acknowledgment:** Within 48 hours
- **Initial assessment:** Within 7 days
- **Fix or mitigation:** Within 30 days (depending on severity)
- **Public disclosure:** Coordinated with reporter, typically after fix is released

---

## Security Considerations

Nawirova runs AI models locally on your device. Key security aspects:

- Models are downloaded from Hugging Face and verified with SHA-256
- No data is sent to external servers after initial setup
- Local server binds only to `127.0.0.1` (localhost)

---

## No Bug Bounty

We do not currently offer a bug bounty program. We appreciate responsible disclosure and will acknowledge contributors in release notes.
