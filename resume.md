# Nestor Wheelock

**Rust Systems Architect & Security Consultant**
Cancun, QR, Mexico | [email] | [phone] | [GitHub URL] | [LinkedIn URL]

---

## Professional Summary

Systems architect and security consultant who designed and built a 78-crate Rust framework spanning 9 architectural layers for enterprise application development. Background in medical research software, infrastructure hardening, and full-stack web development. Builds production systems with compile-time safety guarantees, zero unsafe code, and PostgreSQL-native persistence. Degree in English Literature from Washington University in St. Louis. Legally authorized to work in Mexico; available for remote engagements.

---

## Technical Skills

**Rust & Systems**
Async (Tokio), trait-based architecture, procedural macros, newtype patterns, compile-time invariant enforcement, zero unsafe code policy, WASM (wasm-bindgen, Wasmtime), binary optimization (LTO, strip)

**Web & API**
Axum 0.7, Leptos 0.6 (CSR/SSR), Tower middleware, WebSockets, Askama templates, REST API design, real-time communication

**Database & Persistence**
PostgreSQL with compile-time checked queries (SQLx 0.8), schema migrations, full-text search (Postgres FTS, Meilisearch), Redis caching, SQLite, JSON column types

**Security & Cryptography**
Argon2 password hashing, x25519-dalek key exchange, ChaCha20-Poly1305 encryption, HKDF key derivation, E2E encryption, Web Application Firewall, CVE analysis, WordPress vulnerability assessment, GDPR compliance tooling

**Architecture & Design**
Domain-driven design, layered architecture with compile-time boundary enforcement, state machines, double-entry accounting, multi-tenancy with isolation strategies, event-driven automation, idempotency patterns, soft delete, append-only audit trails

**Infrastructure**
Linux/Unix administration, Docker, S3-compatible object storage, background job queues, rate limiting, health checks, telemetry, structured logging (tracing)

**Other Languages**
Bash, PHP (earlier career), JavaScript/TypeScript (frontend integration)

**Languages**
English (native), working knowledge of Spanish

---

## Selected Projects

### rust-primitives — Enterprise Application Framework

78 composable Rust crates organized across 9 architectural layers (Foundation, Identity, Domain, Content, Infrastructure, Operations, Data, Observability, Platform). Implements the 10 ERP primitives (Party, Catalog, Ledger, Encounter, Decision, Worklog, RBAC, Audit, Singleton, Modules) plus 68 additional domain models. Features compile-time layer dependency enforcement, PostgreSQL-native persistence with checked queries, and a strict no-cross-crate-import rule for loose coupling. Proprietary.

**Key crates include:** multi-tenant auth and RBAC, double-entry ledger, state machine workflows, capacity-based reservations, RFC5545 calendar recurrence, multi-channel notifications, offline-first sync protocol, and a WAF with honeypots.

### Universal Service Daemon (USD)

Cross-platform service management daemon compiled for Linux, FreeBSD, macOS, and Windows. 46+ modules covering security, monitoring, networking, backups, and VM management. WASM module runtime (Wasmtime), WebSocket API, E2E encryption (x25519-dalek + ChaCha20-Poly1305), PTY terminal emulation, and SQLite/PostgreSQL state persistence. Binary-optimized with LTO.

### Plumber — Market Research Automation

Business intelligence tool with HTML scraping (robots.txt-compliant), DNS resolution, TLS/SSL certificate inspection, file hashing (SHA-256, MD5), gzip archival, and real-time WebSocket task delivery. PostgreSQL persistence with JSON column types. Uses 26 rust-primitives crates.

### Scuba Fill Station — Dive Shop Management Platform

Full-stack multi-tenant platform: Axum REST API, Leptos CSR staff portal, customer portal, and marketing site. Domain modeling for dive operations using 45+ primitives crates. Workspace architecture with clean separation between domain, application, API, and UI layers.

### barter-rs — Financial Trading Framework

High-performance trading system with 6 crates: core framework, market data feeds, order execution, exchange integrations, financial instruments, and procedural macros. Decimal-precision arithmetic, cryptographic exchange signatures, WebSocket streaming, and criterion benchmarking.

### WordPress Security Audit Tooling

Methodology for passive reconnaissance of WordPress installations — source code analysis, plugin version enumeration, and CVE correlation — followed by controlled lab replication to verify exploitability. Produces actionable vulnerability reports with remediation steps and automated patching.

### Additional Projects

- **Support System** — Multi-crate ticketing platform (Axum API, core domain, Askama SSR) using 50+ primitives crates
- **WASM Games** — Chess engine, Pong, Moon Buggy, Pitfall — each with Rust backend and WASM frontend
- **Reusable Libraries** — rust-patterns (8 crates), leptos-components, axum-patterns, askama-templates (11 crates)

---

## Professional Experience

### Independent Consultant — Security & Systems Architecture
Mexico (Remote) | 2022 – Present

- Designed and built a 78-crate Rust framework for enterprise application development
- Consult on WordPress security, platform migration, and infrastructure modernization
- Conduct passive vulnerability assessments and deliver verified CVE reports with lab-replicated proof of exploitability
- Build full-stack applications with Axum, Leptos, PostgreSQL, and WASM
- Advise organizations on platform dependency risks, data portability, and multi-tenant architecture

### PADI Scuba Diving Instructor
Cancun, Mexico | 2020 – Present

- Certified PADI instructor leading recreational and training dives
- Client-facing communication, safety management, and logistical coordination in a bilingual environment

### Owner — Computer Repair & IT Services
St. Louis, MO | ~2010 – 2020

- Purchased and operated a retail location, pivoting from software consulting to direct IT services
- Provided hardware repair, network configuration, and technical support to local businesses and consumers
- Managed all business operations including vendor relationships, customer service, and finances

### Co-Founder — Human Research (Consulting & Software Development)
St. Louis, MO | ~2006 – 2010

- Co-founded a consulting firm specializing in custom software for medical research
- Developed PHP-based applications supporting research data collection, management, and reporting
- Managed client relationships with research institutions and government-funded programs

### Customer Service — Quava Cordover Marketing Services, Inc.
St. Louis, MO | ~2005 – 2006

- Entry-level corporate role; met future business partner, leading to co-founding of software venture

---

## Education

**Washington University in St. Louis**
Bachelor of Arts — English Literature, Minor in History
Coursework in Philosophy (ethics and problem-solving frameworks)

---

## Certifications

- CompTIA Security+ (planned / in progress)
- PADI Scuba Diving Instructor Certification

---

## Approach

Solutions are built on a 78-crate Rust framework with compile-time enforced layer boundaries and strict separation of concerns. Every domain model follows established patterns: UUID primary keys, soft delete, append-only audit trails, and typed errors. Clients retain full ownership and portability of their data. External developers interact through a documented API, maintaining security and architectural integrity. Zero unsafe code. All invariants enforced in both Rust's type system and PostgreSQL constraints.
