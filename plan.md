# kiki_ssh — Research & Implementation Plan

Goal
- Research how SSH works (protocol, implementation ecosystem, attack surface), identify missing features, and produce a secure feature roadmap and implementation plan for a new project named `kiki_ssh`.

Initial findings (workspace scan)
- No git repo detected at workspace root.
- Notable entries: a personal ~/.ssh directory and a project folder `mcp_ssh_for_wnogui` — suggests SSH-related experiments may exist.

Scope (per your choice)
- Protocol deep dive: RFCs, wire protocol, key exchange, algorithms, extensions, and known vulnerabilities.
- Feature roadmap & implementation plan: a prioritized list of features to add for security and usability, and an implementation approach for `kiki_ssh`.

Approach
1. Research phase
   - Read and summarize relevant RFCs (SSH-TRANS, AUTH, CONNECTION, etc.).
   - Survey major implementations (OpenSSH, libssh, libssh2, Dropbear) — differences, strengths, weaknesses.
   - Inventory known vulnerabilities and common misconfigurations.
2. Design & threat modeling
   - Define threat model and attacker capabilities.
   - Identify missing features and security hardening points.
3. Feature prioritization & roadmap
   - Propose features (e.g., modern KEX, post-quantum key options, robust key management, hardware-backed keys, MFA, session attestation, audit logging, minimal privileged helpers, sandboxing, formal verification where practical).
   - Prioritize by security impact, implementation effort, compatibility risk.
4. Prototype & implementation plan
   - Choose implementation language and libraries (recommend Rust or Go for safety).
   - Architecture: transport layer, auth, server agent, client, key management, plugins.
   - Define tests, CI, fuzzing, and review processes.
5. Validation & rollout
   - Pen-testing, code review, CI, reference deployment, documentation, migration path from OpenSSH.

Deliverables
- Research summary (RFCs + implementations)
- Threat model and gap analysis
- Prioritized feature roadmap with safety/cost tradeoffs
- Architecture and implementation plan for `kiki_ssh`
- Todo list and milestones

Todos (high level)
- research-rfcs: Summarize SSH RFCs and protocol details.
- survey-implementations: Compare OpenSSH, libssh, libssh2, Dropbear.
- inventory-vulns: Catalog common vulnerabilities and misconfigurations.
- threat-model: Produce threat model and attacker capabilities.
- propose-features: Produce prioritized feature list for kiki_ssh.
- select-tech: Decide implementation language, libraries, and toolchain.
- design-architecture: High-level architecture and module breakdown.
- prototype-core: Build minimal prototype (KEX + auth) proof-of-concept.
- security-testing: Define tests, fuzzing, CI, and audit steps.
- docs-and-roadmap: Write documentation and release roadmap.

Notes and open questions
- Confirm desired implementation language and target platforms.
- Confirm whether backward compatibility with OpenSSH is required.
- Confirm ownership of existing `mcp_ssh_for_wnogui` (should it be imported/reused?).

Next step options
- Start by running an in-repo audit for SSH-related code and folders (recommended if code exists).
- Or begin protocol research and RFC summarization immediately.

Decisions made
- Implementation language: Rust (chosen). Rationale: memory-safety, mature crypto crates, good performance and ecosystem for secure system software.
- Target platforms: Linux (x86_64, ARM) initially; Windows/macOS later.
- Compatibility: Aim for interoperability with OpenSSH auth methods and authorized_keys format; not a direct fork.

Milestones (estimates)
1) Research (2 weeks): research-rfcs, survey-implementations, inventory-vulns.
2) Design (2 weeks): threat-model, propose-features, design-architecture.
3) Prototype (4 weeks): prototype-core, security-testing.
4) Release (2 weeks): docs-and-roadmap, packaging, CI.

Next automated actions
- Marked 'select-tech' as done and started 'research-rfcs' and 'survey-implementations' (in_progress).

Decisions made
- Implementation language: Rust (chosen). Rationale: memory-safety, mature crypto crates, good performance and ecosystem for secure system software.
- Target platforms: Linux (x86_64, ARM) initially; Windows/macOS later.
- Compatibility: Aim for interoperability with OpenSSH auth methods and authorized_keys format; not a direct fork.

Milestones (estimates)
1) Research (2 weeks): research-rfcs, survey-implementations, inventory-vulns.
2) Design (2 weeks): threat-model, propose-features, design-architecture.
3) Prototype (4 weeks): prototype-core, security-testing.
4) Release (2 weeks): docs-and-roadmap, packaging, CI.

Full Project Plan — kiki_ssh

1. Executive summary
kiki_ssh is a secure, modern SSH implementation focusing on safety, auditability, and deployable hardening defaults. Built in Rust, kiki_ssh will prioritize memory safety, modular cryptography, and a minimal trusted computing base while maintaining compatibility with common SSH clients and servers where feasible.

2. Goals
- Implement a secure SSH server and client stack suitable for production use.
- Provide stronger defaults and mitigations against common misconfigurations and attacks.
- Enable hardware-backed keys, multi-factor authentication, and strong auditing.
- Design for extensibility: plugins for auth, session policies, and transport extensions.

3. Scope
- Protocol support: implement core SSH-2 features (transport, userauth, connection), common auth mechanisms (publickey, password, hostbased), and SFTP subsystem later.
- Interop: support OpenSSH-compatible key formats (authorized_keys) and basic control channel semantics to interoperate with OpenSSH clients.
- Exclusions: legacy SSH-1, binary compatibility with OpenSSH server internals, and complex PAM integrations initially.

4. Deliverables
- RFC and implementation survey documents
- Threat model and attack surface analysis
- Prioritized feature roadmap and success criteria
- High-level architecture and module breakdown
- Prototype implementing KEX + publickey auth and minimal shell subsystem
- CI pipeline with static analysis, fuzzing, and unit/integration tests
- Documentation: developer guide, admin guide, migration notes
- Release artifacts and packaging

5. Timeline & milestones (12-week plan)
Week 1–2: Research
- Summarize SSH RFCs, survey existing implementations, catalog CVEs and common misconfigurations
- Deliverable: Research summary
Week 3–4: Design
- Threat modeling, requirements, high-level architecture, API surfaces
- Deliverable: Design doc and prioritized roadmap
Week 5–8: Prototype
- Implement a minimal server and client in Rust: TCP listener, KEX (ECDH), userauth publickey, session channel skeleton
- Add unit tests, integration tests against OpenSSH client/server
- Deliverable: Prototype and test suite
Week 9–10: Security testing
- Static analysis, dependency audit, fuzzing (cargo-fuzz), targeted pentest
- Deliverable: Security test reports and mitigations
Week 11–12: Docs & release prep
- Packaging, docs, migration guide, CI release workflow, initial demo deployment
- Deliverable: v0.1 release candidate

6. Feature list (prioritized)
Core (MVP)
- Secure defaults (strong ciphers, disabled weak algorithms)
- KEX: ECDH (curve25519) and modern KEXs
- Publickey authentication supporting OpenSSH formats
- Simple shell/exec subsystem and command forwarding
- Robust logging and audit hooks

High value (v1)
- Hardware-backed keys (YubiKey/PKCS#11 support)
- MFA: time-based OTP integration and optional authentication flow
- Session recording (selective, privacy-aware)
- Fine-grained host/user policy engine
- SFTP support (limited scope)

Advanced (v2+)
- Post-quantum hybrid KEX options (research + opt-in)
- Formal verification of critical parsers
- Sandboxed helpers (drop privileges, seccomp/apparmor integration)
- Enterprise features: LDAP, RBAC plugins, centralized audit

7. Threat model (summary)
Assets: user credentials, host private keys, session data, audit logs.
Adversaries: remote unauthenticated attackers, authenticated but malicious users, local privilege escalators, supply-chain attackers.
Threats & mitigations:
- Memory corruption → mitigate via Rust, minimal unsafe code, fuzzing
- Weak algorithms → default to modern ciphers; reject legacy KEX/ciphers
- Credential theft → support hardware-backed keys, strong key storage, minimize exposures
- Brute-force/password guessing → rate-limiting, fail2ban-like hooks, MFA
- Lateral movement via misconfig → strict session policies, chroot/sandboxing for subsystems

8. Architecture & components
- Transport layer: connection listener, TLS-like framing per RFC, KEX implementations, algorithm negotiation
- Crypto layer: abstracted crypto provider; defaults using vetted crates (ring/openssl wrappers as needed)
- Auth layer: pluggable auth backends (publickey, password, OTP, PKCS#11)
- Session manager: channel multiplexing, pty allocation, exec/shell handling
- Key management: config for host keys, authorized_keys parsing, key rotation helpers
- Admin & logging: structured logs, audit trail, metrics endpoint
- Agent/plugin API: for policy, auth, and session hooks

9. Implementation details & tech choices
- Language: Rust
- Async runtime: tokio
- Crypto: use well-audited crates (ring, or bindings to libsodium/openssl) and limit crypto surface to well-tested modules
- Parsing: safe parsers with unit tests and fuzz targets
- Build & CI: GitHub Actions, cargo test, clippy, cargo-audit, cargo-fuzz
- Testing: interop tests with OpenSSH, conformance tests for RFC wire formats

10. Quality, security & testing
- Static analysis: clippy, miri for unsafe checks in CI on select matrix
- Dependency checks: cargo-audit on every run
- Fuzzing: corpus seeds from real SSH captures (sanitized) and RFC examples
- Pen test: threat-specific test plan (auth bypass, replay, malformed frames)
- Code review and security sign-off for all unsafe code

11. Compliance, licensing & governance
- License: permissive (BSD-2 or MIT) recommended for adoption; consider dual-licensing if needed
- Contributor guidelines, code of conduct, security disclosure process
- Responsible disclosure and CVE process documentation

12. Ops, deployment & migration
- Packaging: distro packages (deb/rpm), static releases, container images
- Defaults: secure config shipped; admin opt-in for weaker options
- Migration: tools for converting authorized_keys, host key importers, and compatibility layers

13. Metrics & success criteria
- Interop: pass 95% of typical OpenSSH client workflows in tests
- Security: zero critical CVEs at v0.1; coverage reports and fuzzing findings reduced to acceptable levels
- Performance: comparable latency/throughput to Go-based lightweight servers in common scenarios
- Adoption: usable by admins with clear migration docs

14. Risks & mitigations
- Cryptography correctness: use vetted libraries, avoid home-grown crypto
- Interop edge cases: maintain OpenSSH-compatible parsing and test corpus
- Resource constraints: modular design to allow lightweight builds
- Supply-chain risk: pinned dependencies, reproducible builds

15. Team & roles (if solo, mapped to milestones)
- Research lead: RFCs, implementation survey, threat model
- Design lead: architecture, feature prioritization
- Implementation lead: core prototype, tests
- Security lead: fuzzing, audits, pentest coordination
- Docs & release: packaging, admin guides, CI

16. Next steps (immediate)
- Finalize and approve this plan (done)
- Begin Research sprint: research-rfcs and survey-implementations (in_progress)
- Create skeleton repo and initial README when ready (no code yet per your request)

Appendix: useful references to fetch during research
- SSH RFCs: RFC 4251--4254 (SSH protocol architecture, transport, user authentication, connection protocol)
- Implementation projects to review: OpenSSH, libssh, libssh2, Dropbear, rust-ssh projects (for examples)
- Tools: cargo-fuzz, cargo-audit, clippy, bandit (for scripting checks)

Acceptance: this plan saved as the authoritative roadmap for kiki_ssh. Update as research reveals new constraints.

Decisions made
- Implementation language: Rust (chosen). Rationale: memory-safety, async ecosystem, and strong tooling for secure system software.
- Compatibility: Strong compatibility with OpenSSH primitives (certificates, forced commands, authorized_keys), no custom cryptography.
- Scope reframed: kiki_ssh becomes an Agent SSH System — a secure access layer for AI agents called the Agent Access Gateway, with box-side wrappers, a policy engine, and strict auditing.

Agent SSH System — Revised Full Project Plan (kiki_ssh)

Executive summary
kiki_ssh is not merely another SSH implementation. It is a secure Agent SSH System that mediates and scopes AI agent access to hosts using proven SSH primitives (OpenSSH certs, forced commands) while adding a control plane, policy enforcement, short-lived credentials, and full auditability. The design prevents agents from receiving unrestricted human-style SSH access and enables instant revocation, approval workflows, and least-privilege sessions.

Architecture overview
1) Agent Access Gateway (Control Plane)
- Central Rust service (async tokio) that authenticates agents (via signed attestation / agent identity provider).
- Issues short-lived OpenSSH user certificates or temporary tokens to agent clients after policy evaluation.
- Records agent metadata: agent id, human owner, task id/prompt, repo/project, requested host(s), requested commands, and rationale.
- Enforces pre-conditions: policy checks, risk scoring, approval requirements.
- Provides audit stream, metrics, and an API (gRPC/HTTP) for agents and admin dashboards.

2) Box-side Agent SSH Daemon / Wrapper
- Runs alongside or co-located with OpenSSH server (uses authorized_keys/forced-commands or sshd Match configuration).
- Accepts certificate-based auth from gateway-issued certs and maps certificate principals to scoped runtime permissions.
- Implements forced-command wrappers that implement: allowlist of commands, contextual environment (task metadata), path restrictions, chroot/sandbox options, and robust logging/audit hooks.
- Ensures default denial: agents never get an unrestricted interactive root shell.

3) Policy Engine
- Centralized policy datastore with declarative policies (YAML/JSON) describing allowed principals, hosts, tasks, time windows, command allowlists, and required approval levels.
- Policy evaluation occurs in the Gateway before issuing certs and optionally on the box-side wrapper for defense-in-depth (re-checking token and task context).
- Support dry-run mode and human-approval mode with audit trail.

4) Session Model & Lifecycle
- Session request: agent calls Gateway with identity, human owner, task prompt, target host, requested commands/paths.
- Gateway evaluates policy -> issues short-lived cert (e.g., valid 5–30 minutes) or denies/requests approval.
- Agent connects to host using cert; box wrapper enforces runtime restrictions and streams audit logs back to Gateway.
- Session metadata: agent identity, human owner, task/prompt, requested vs. granted permissions, time window, host, command list, session id.
- Revocation: Gateway can immediately revoke a cert (CRL / short lifetime + active kill switch that instructs box-side wrapper to terminate session by session id).

Security model & principles
- No long-lived agent SSH keys; only short-lived certs and tokens.
- No shared credentials or passwords for agents.
- Principle of least privilege: default-deny, minimal allowed commands and file paths.
- Defense-in-depth: Gateway + box-side enforcement + audit verification.
- Use OpenSSH compatibility — do not invent crypto.
- Strong logging: per-command auditing, session recording (optional), immutable audit trails.
- Human-in-the-loop for dangerous operations (approval workflows, MFA for approvals).

Threat model (summary)
Assets: host integrity, credentials, sensitive data on host, audit logs, agent identity.
Adversaries: compromised agent, malicious agent authors, compromised control plane, insider admins, supply-chain.
Key threats & mitigations:
- Misuse of agent credentials: short-lived certs, limited scope, immediate revocation.
- Escalation to root: forced commands, path allowlists, chroot/sandbox, privilege drop.
- Eavesdropping of sessions: encrypted SSH, audit log shipping to central store.
- Control plane compromise: segregated admin roles, signed attestation for hosts, offline key backups.
- Supply-chain risk: pinned dependencies, reproducible builds, signed releases.

MVP roadmap (realistic phased plan)
MVP 1 — Control Plane Core
- Rust control-plane service with agent/host registry
- Declarative policy format (YAML/JSON) and evaluator
- Issuing short-lived OpenSSH user certificates (via ssh-keygen/ssh-cert utilities or using an OpenSSH CA key)
- Simple API to request access; record access requests and responses (audit log)
Deliverables: Gateway prototype, policy schema, API, basic audit log

MVP 2 — Box-side Enforcement
- Design and deploy a forced-command wrapper for OpenSSH authorized_keys entries or sshd Match + ForceCommand
- Enforce command allowlist and path restrictions; populate environment with session metadata
- Basic audit logging (per-command logs) streamed to Gateway or pushed periodically
Deliverables: box wrapper scripts/daemon, host registration, end-to-end request -> session flow

MVP 3 — Human Approval & Dashboard
- Approval workflow: requests that exceed policy risk require manual approve; notifications (email/Slack)
- Web/TUI dashboard showing active sessions, request metadata, and audit logs
- Revoke button to terminate session immediately
Deliverables: dashboard, approval flow, revoke implementation

MVP 4 — Agent-friendly Interfaces & SDK
- Provide SDKs (Rust, Python) and CLI for agents to request scoped operations (task-run API) that perform safe operations without a raw shell
- Add SSH fallback only when required; prefer high-level RPC-like operations for common agent needs (e.g., fetch logs, run a single command limited by policy)
Deliverables: SDKs, higher-level agent APIs, integration examples

MVP 5 — Advanced Security & Observability
- Session recording and replay for forensic review
- Anomaly detection (flag unusual command usage, exfiltration patterns)
- Policy suggestion engine (ML-assisted suggestions based on usage patterns)
- PKCS#11 / hardware-backed host keys and optional YubiKey support for admin approval
Deliverables: recording, analytics, optional hardware-backed checks

Implementation tasks (concrete)
- Control plane
  - agent registry, host registry, CA key management
  - access request API, policy evaluator, audit store
  - cert issuance pipeline (integration with OpenSSH CA or ssh-keygen)
  - revoke/kill-switch API
- Host-side
  - wrapper binary (Rust) or minimal shell wrapper integration with OpenSSH
  - session lifecycle management and logging client that connects to Gateway
  - sandboxing helpers (unshare, seccomp profiles, chroot templates)
- Policy
  - authoring format, editor tools, test harness (policy simulator / dry-run)
  - pre-built policy templates (read-only logs, maintenance tasks, deploy-only)
- UX & admin
  - dashboard, session list, approval flows, audit query interface
  - integrations (Slack, email, PagerDuty)
- Security & testing
  - CI with clippy, cargo-audit, cargo-fuzz
  - interop tests vs OpenSSH client/server
  - pentest checklist and threat-specific tests

Why an Agent Access Gateway is better than giving AI agents normal SSH keys
- Least privilege: Gateway issues scoped, short-lived certs tailored to a task; normal keys give broad, persistent access.
- Auditable: every request is recorded with task/prompt metadata — normal keys lack this provenance.
- Revocation & control: instant revocation and kill-switch; human approval required for dangerous actions.
- Policy enforcement: pre-checks prevent dangerous commands from running; host-side wrappers re-check for defense-in-depth.
- Safer workflows: agents can be given high-level APIs for tasks (e.g., fetch log lines), reducing the need for raw shells.
- Compliance: clear audit trails and approval records help with compliance and incident response.

Privacy & governance
- Record minimal necessary metadata; redact sensitive content from logs where appropriate.
- Provide opt-in session recording with access controls for who can view replays.
- Define responsible disclosure and admin governance policies.

Milestones, timeline & resourcing
- Weeks 1–2: Finalize design; policy schema; CA/key management approach; host wrapper design
- Weeks 3–6: Build MVP1 control-plane + cert issuance + host registration
- Weeks 7–10: Build MVP2 box-side wrapper and end-to-end flow
- Weeks 11–14: Build MVP3 approval workflows and dashboard
- Weeks 15–20: Build MVP4 SDK/API and hardened deployments
- Weeks 21–26: Build MVP5 advanced features (recording, analytics, hardware integration)

Acceptance criteria & success metrics
- Agents cannot obtain interactive root shells by default
- All agent accesses include a recorded request (agent id, human owner, task prompt)
- Cert issuance and revocation tested end-to-end
- Host-side wrapper enforces command/path allowlists in majority of test cases
- Dashboard displays active sessions within <2s of creation

Immediate next steps (no code yet)
- Approve this revised plan
- Finalize policy schema examples for common agent tasks
- Design OpenSSH CA integration approach (signing workflow)

Plan saved to session plan file and tracked in todos. Update tasks when ready to start implementation.

-- end --
