# System Behavior and Trust Reference

## Purpose

This document provides a **human‑readable reference** for understanding how the
certificate renewal proxy behaves, why it behaves that way, and what design
intent underpins its decisions.

It exists to explain:
- trust boundaries
- expected behavior
- intentional failure cases
- design tradeoffs

This document is **descriptive**, not normative.

Normative rules, APIs, and enforcement are defined elsewhere.

---

## What This System Is For

The certificate renewal proxy exists to:

- automate certificate issuance and renewal
- for non‑Windows infrastructure
- across heterogeneous environments
- without weakening domain or identity trust boundaries

Typical use cases include:
- SIP / SBC TLS certificates
- network appliances
- infrastructure services that cannot act as ACME clients

The system is deliberately conservative.  
Automation must never expand trust implicitly.

---

## Core Behavioral Principles

### 1. Domains Are Trust Boundaries

Each DNS domain (or domain pattern) represents a **separate trust authority**.

Certificates may only be issued under a domain if there is an explicit
Domain Trust Profile authorizing that domain.

If a domain is not explicitly covered, issuance fails.

This prevents:
- accidental certificate sprawl
- silent authority expansion
- misconfigured automation

---

### 2. Authentication Is Not Authorization

A client may:
- authenticate successfully
- but still be rejected

Authentication establishes *who* is calling.  
Authorization establishes *what they are allowed to do*.

Authorization is always evaluated in the context of a specific domain.

This prevents:
- reuse of credentials across domains
- lateral movement
- “works once, works everywhere” mistakes

---

### 3. Ambiguity Is Treated as Failure

If the system cannot determine:
- a single authoritative Domain Trust Profile
- a single policy context
- a single trust authority

…the request is rejected.

The system never:
- guesses
- falls back
- partially approves requests

This is intentional.

Operational inconvenience is preferable to silent security failure.

---

## Why Requests Fail (By Design)

Many rejected requests are **intentional**, not errors.

Examples:

- A request includes SANs from multiple domains  
  → Rejected to prevent cross‑domain certificates

- A client is authenticated but not authorized  
  → Rejected to prevent identity abuse

- No Domain Trust Profile matches a SAN  
  → Rejected to prevent implicit authority

- Wildcards are requested where disallowed  
  → Rejected to prevent trust amplification

These failures are what make the system safe.

---

## The Proxy’s Own Trust Model

The proxy is **not a privileged internal authority**.

Instead:
- it authenticates like any other client
- it enrolls for its own certificate via the same API
- it is authorized via a dedicated internal trust profile

This prevents:
- hidden root‑like behavior
- permanent bootstrap credentials
- unreviewable trust paths

If the proxy cannot enroll itself correctly, it must not operate.

---

## DNS Behavior

The proxy never “owns” DNS.

It is permitted to modify DNS only when:
- explicitly authorized by a Domain Trust Profile
- using scoped credentials
- for `_acme-challenge` records only

This allows:
- internal DNS
- external DNS providers
- mixed environments

without creating DNS‑level privilege escalation.

---

## Relationship to Other Documentation

This document should be read alongside:

- Architecture documents (what *must* happen)
- API contracts (what clients can expect)
- Acceptance tests (what *must* work)
- Negative tests (what *must never happen*)
- Examples (how this looks in practice)

If there is a conflict:
- architecture and tests take precedence
- this document explains intent, not enforcement

---

## How to Use This Document

This document is intended for:
- onboarding new engineers
- explaining behavior to security teams
- design reviews
- incident retrospectives
- future maintainers

It is not intended to:
- define APIs
- replace tests
- override policy

---

## Summary

The certificate renewal proxy is intentionally strict.

It prefers:
- explicit trust
- clear authority
- deterministic behavior
- safe failure

over convenience or implicit automation.

This reference document explains *why*.
``