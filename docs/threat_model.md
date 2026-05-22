# Threat Model & Policy Schema — kiki_ssh (Agent SSH System)

## Overview
This document defines the threat model and a policy schema for kiki_ssh, an Agent SSH System that provides scoped, auditable SSH access to AI agents via an Agent Access Gateway and host-side enforcement.

## Goals
- Ensure least privilege access for AI agents.
- Make agent actions auditable and reversible.
- Prevent long-lived or broad SSH credentials for agents.
- Provide human approval for high-risk operations.

## Key assets
- Host integrity and sensitive data stored on hosts
- Host private keys and CA signing key(s)
- Agent identities and session tokens/certs
- Audit logs and session recordings

## Adversaries
- Compromised agent (malicious code or misconfigured agent)
- Compromised control plane (gateway)
- Malicious or negligent human operator
- Supply-chain compromise of dependencies

## Attack surface
- Certificate issuance and CA key handling
- Network layer between agent, gateway, and host
- Host wrapper parsing and command enforcement
- Audit log integrity and exfiltration channels

## Mitigations
- Short-lived certificates (5–30 minutes) and immediate revocation via kill-switch
- Defense-in-depth: gateway policy + host-side re-checks
- Minimal trusted code on hosts; implement boxed wrapper in Rust with minimal unsafe
- Centralized immutable audit logs; sign or append-only storage
- Hardened CI, pinned dependencies, reproducible builds

## Policy principles
- Default deny: deny any action not explicitly allowed by policy
- Scope by task: policies include task-id, human-owner, and reason
- Time-boxed: policies specify max-duration and expiry
- Human approval for risky actions
- Dry-run support for testing policies without granting perms

## Policy schema (YAML)

example_policy:
  id: "read-logs"
  description: "Allow agent to read application logs for task debugging"
  principals:
    - agent: "ci-bot-123"
  hosts:
    - "prod-app-01.example.net"
  commands:
    allow:
      - "journalctl -u myapp --no-pager"
      - "cat /var/log/myapp/*.log"
  paths:
    allow:
      - "/var/log/myapp/*"
  max_duration: 30m
  require_approval: false
  dry_run: false

Fields explained:
- principals: list of agent identities or groups
- hosts: FQDNs or tags for target hosts
- commands: explicit allow/deny lists (exact or regex)
- paths: allowed filesystem path globs
- max_duration: session lifetime
- require_approval: if true, gateway creates a pending request for human approval
- dry_run: if true, evaluate policy but do not grant certs

## Approval workflow
- Agent requests access with metadata (agent id, owner, task id, prompt)
- Gateway evaluates policy and if require_approval is true, creates a pending approval
- Approver can grant with optional scoped adjustments; approval is logged
- Gateway issues short-lived cert only after approval

## Session revocation & kill-switch
- Gateway can revoke by adding session id to an active revoke table and instructing host wrapper to terminate session by id
- Hosts poll or maintain a control channel for rapid revoke notifications

## Host-side hardening
- Use forced-command wrappers and environment variables containing session metadata
- Run commands with dropped privileges where possible, chroot, and seccomp filters
- Stream logs and session metadata to central audit store (TLS + mTLS)

## Testing & validation
- Unit tests for policy evaluator and parser
- Fuzzing the host wrapper command parser
- Integration tests: request->issue cert->connect->enforce
- Pen-test plan for common escalation paths

## Next steps
1. Finalize policy field types and validation rules
2. Implement policy simulator for dry-run testing
3. Draft host-wrapper API for session control and revoke

---
Generated from plan: /home/kihi2024/.copilot/session-state/aa93c3f0-23c1-4612-bdab-1d3b0178d990/plan.md
