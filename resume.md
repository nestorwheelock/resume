# Nestor Wheelock

**Rust Systems Architect & Security Consultant**
Cancun, QR, Mexico | [email] | [phone] | [GitHub URL] | [LinkedIn URL]

---

## Professional Summary

Systems architect and security consultant who designed and built a 78-crate Rust framework spanning 9 architectural layers for enterprise application development. Prototyped domain models across 21 Django applications — extracting reusable packages for multi-tenancy, accounting, scheduling, and CRM — then rewrote the entire ecosystem in Rust for compile-time safety, zero unsafe code, and PostgreSQL-native persistence. Degree in English Literature from Washington University in St. Louis, combined with 20 years of reading and writing software, provides a distinct edge in LLM-driven development: precise plain-language specification, effective prompt engineering, and clear technical documentation as the human in the loop. Legally authorized to work in Mexico; available for remote engagements.

---

## Technical Skills

**Rust & Systems**
Async (Tokio), trait-based architecture, procedural macros, newtype patterns, compile-time invariant enforcement, zero unsafe code policy, WASM (wasm-bindgen, Wasmtime), binary optimization (LTO, strip)

**Python & Django**
Django 5.x/6.x, Django REST Framework, Celery/Redis task queues, Django Channels (WebSockets), Wagtail CMS, HTMX, pytest (87%+ coverage on production projects), extractable pip-installable package architecture, multi-tenant SaaS patterns, Stripe integration, AI/LLM integration via OpenRouter

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

**LLM-Driven Development**
Precise plain-language specification for AI-assisted code generation, prompt engineering, human-in-the-loop workflow design, intent-driven automation, technical documentation informed by a liberal arts background and decades of reading open-source software

**Other Languages**
Bash, PHP (earlier career), JavaScript/TypeScript, Flask

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

### Django Prototyping Ecosystem — 21 Applications, 23 Extractable Packages

Rapid-prototyped domain models across 21 Django applications to validate business logic before rewriting in Rust. Extracted 23 reusable pip-installable packages (django-basemodels, django-parties, django-accounting, django-appointments, django-inventory, django-multilingual, and more) that mirror the architecture of rust-primitives. Key applications include:

- **FleetFlow** — Multi-tenant car rental SaaS with AI-powered OCR (license/insurance scanning, damage detection), Celery task queues, 368 tests at 87% coverage
- **Procurement Scraper** — Distributed government procurement intelligence with self-hosted instances, embedded AI processing, and bidirectional sync to a central cache
- **Bruno Health Tracker** — Veterinary cancer treatment quality-of-life tracker using validated clinical assessment instruments (CBPI, CORQ, VCOG-CTCAE)
- **Buceo Feliz** — Dive operations platform with digital agreements, e-signatures, medical questionnaires, and invoicing
- **Business Hub** — Integration test harness demonstrating all 23 extracted packages running together

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

**Washington University in St. Louis** (Top 10, U.S. News at time of graduation)
Bachelor of Arts — English Literature, Minor in History
Coursework in Philosophy (ethics and problem-solving frameworks)

---

## Certifications

- CompTIA Security+ (planned / in progress)
- PADI Scuba Diving Instructor Certification

---

## Approach

Solutions are built on a 78-crate Rust framework with compile-time enforced layer boundaries and strict separation of concerns. Every domain model follows established patterns: UUID primary keys, soft delete, append-only audit trails, and typed errors. Clients retain full ownership and portability of their data. External developers interact through a documented API, maintaining security and architectural integrity. Zero unsafe code. All invariants enforced in both Rust's type system and PostgreSQL constraints.
