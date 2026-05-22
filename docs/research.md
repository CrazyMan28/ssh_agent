# Research: SSH RFCs & Implementation Survey

This document completes the research deliverable for Issue #1:
- Summarize RFCs 4251-4254 and related specs.
- Survey OpenSSH, libssh, libssh2, Dropbear, and Rust SSH projects.
- Capture design implications for `ssh_agent` / `kiki_ssh`.

## 1) RFC summaries (SSH-2 core + related)

### RFC 4251 — SSH Protocol Architecture
- Defines SSH-2 architecture and terminology.
- Splits protocol into three layers:
  1. Transport (confidentiality, integrity, server auth, optional compression)
  2. User Authentication
  3. Connection Protocol (channels, multiplexing sessions over one transport)
- Sets algorithm negotiation model and extensibility approach.

### RFC 4252 — SSH Authentication Protocol
- Defines user authentication over the protected transport.
- Core methods: `publickey`, `password`, `hostbased`; supports method negotiation and partial success.
- Security implications:
  - Public key auth should be preferred.
  - Password auth should be rate-limited and optionally disabled by policy.

### RFC 4253 — SSH Transport Layer Protocol
- Defines wire framing, version exchange, key exchange (KEX), rekeying, encryption, and MAC handling.
- Security-critical points:
  - Strict algorithm negotiation and downgrade resistance are required.
  - Rekeying limits should be enforced by bytes/time thresholds.
  - Weak or legacy algorithms must be disabled by default.

### RFC 4254 — SSH Connection Protocol
- Defines channels and requests (`session`, `exec`, `shell`, `subsystem`, forwarding).
- Enables multiplexed channels over one authenticated transport.
- Security implications:
  - Channel requests need explicit policy checks.
  - PTY, port-forwarding, and env requests are high-risk and should default to deny.

### Related RFCs and extensions (high impact)
- RFC 5656: ECC support for SSH.
- RFC 8332: RSA SHA-2 signatures (`rsa-sha2-256`, `rsa-sha2-512`) replacing SHA-1 usage.
- RFC 8308: Extension negotiation (`ext-info`) to evolve behavior safely.
- RFC 8709: Ed25519/Ed448 key algorithms.
- RFC 9142: Strict key exchange hardening guidance and interoperability considerations.

## 2) Implementation survey

| Project | Language | Strengths | Tradeoffs / Risks | Relevance to `kiki_ssh` |
|---|---|---|---|---|
| OpenSSH | C | Most deployed, strongest interop baseline, mature hardening | C memory safety burden, large legacy surface | Primary compatibility reference and behavior baseline |
| libssh | C | Full protocol library, server/client features, embeddable | Historic parser/auth vulnerabilities in ecosystem context; careful upgrade posture needed | Good API ideas for modular auth/session components |
| libssh2 | C | Lightweight client-side library, broad platform use | Narrower scope, C safety concerns remain | Useful for client interoperability tests |
| Dropbear | C | Small footprint, embedded-friendly, pragmatic defaults | Fewer enterprise features, still C memory safety exposure | Useful model for minimal TCB and constrained environments |
| thrussh / russh (Rust) | Rust | Memory safety, async-friendly design, easier parser safety | Smaller ecosystem, lower deployment maturity than OpenSSH | Strong patterns for Rust-first transport/auth/channel modules |
| WezTerm SSH / assorted Rust crates | Rust | Practical Rust SSH usage and ecosystem examples | Not a single canonical full-stack SSH daemon baseline | Reference implementations for protocol handling patterns |

## 3) Common vulnerability themes from ecosystem history

- Parser bugs and memory corruption in packet handling.
- Authentication bypasses due to state-machine edge cases.
- Algorithm downgrade / weak-crypto acceptance when legacy compatibility is over-prioritized.
- Unsafe defaults for forwarding, environment passthrough, and unrestricted command execution.
- Logging/audit gaps that make incident response difficult.

## 4) Design implications for `ssh_agent`

### Protocol and crypto
- Implement strict algorithm allowlists with modern defaults:
  - Curve25519-class KEX
  - AEAD or strong encrypt+MAC combinations
  - RSA SHA-2 and Ed25519 host/user keys
- Disable weak/legacy algorithms by default; require explicit opt-in with audit markers.
- Enforce rekeying thresholds and extension negotiation handling.

### Authentication and authorization
- Prefer short-lived certificate/public-key flows over password auth.
- Model auth and channel handling as explicit state machines to prevent bypass.
- Keep host-side policy enforcement independent from gateway decisions (defense-in-depth).

### Connection/channel policy
- Default-deny risky channel types and requests:
  - Port forwarding
  - Arbitrary environment injection
  - Unscoped subsystems
- Require policy-scoped allowlists for command/path access.

### Implementation strategy
- Rust-first core with minimal unsafe code paths.
- Fuzz packet parsers and state transitions early.
- Build interop tests against OpenSSH for each milestone.

## 5) Recommended follow-on tasks

1. Convert these findings into concrete protocol/auth requirements in issue #2 and issue #3 design docs.
2. Define mandatory algorithm policy and compatibility matrix (OpenSSH interop targets).
3. Add parser/state-machine fuzzing targets as soon as transport parsing exists.

---

Issue linkage: https://github.com/CrazyMan28/ssh_agent/issues/1
