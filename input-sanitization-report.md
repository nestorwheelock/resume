# Input Sanitization & Security Architecture Report

## Overview

This document describes the input sanitization, validation, and security patterns implemented across the rust-primitives framework, the Universal Service Daemon (USD), and Axum-based API applications. The architecture follows a defense-in-depth strategy with seven distinct validation layers, from network edge to database constraints.

---

## Layered Sanitization Model

```
Request arrives
    → WAF (regex pattern detection, rate limiting, honeypots)
        → Axum middleware (tenant validation, auth, mTLS)
            → Serde deserialization (type-safe boundary)
                → Validator crate (#[validate] attributes)
                    → Service layer (business rule checks, permissions)
                        → SQLx (compile-time checked, parameterized queries)
                            → PostgreSQL (database constraints, CHECK/UNIQUE/FK)
```

Each layer catches a different class of attack. No single layer is trusted to be sufficient on its own.

---

## Layer 1: Web Application Firewall (rust-primitives)

**Crate:** `primitives-waf` (`crates/infrastructure/waf/`)

### Pattern Detection

The WAF runs regex-based detection across all inbound request bodies, query strings, and headers.

**SQL Injection** — 18 regex patterns covering:
- UNION SELECT, INSERT INTO, DELETE FROM, DROP TABLE
- SQL comments (`--`, `/* */`)
- information_schema enumeration
- EXEC/EXECUTE calls
- Stacked queries via semicolons

**Cross-Site Scripting (XSS)** — 15+ patterns covering:
- `<script>` tags and event handlers (`onerror`, `onload`, etc.)
- `javascript:` protocol in attributes
- Base64-encoded payloads
- `document.*` and `window.*` manipulation
- SVG, iframe, and object tag injection

**Path Traversal** — patterns covering:
- `../` sequences
- URL-encoded variants (`%2e%2e%2f`)
- Double encoding
- Null byte injection (`%00`)
- Absolute path access (`/etc/passwd`, `c:\windows`)

**Data Leak Detection** — outbound scanning for:
- **SSN**: Luhn-style validation with area/group/serial format checks, rejects known invalid patterns (666, 900+, all zeros)
- **Credit Cards**: Visa, Mastercard, Amex, Discover with Luhn algorithm checksum verification
- **API Keys**: AWS key patterns (`AKIA`, `A3T`), generic bearer tokens

### Rate Limiting

Token bucket algorithm with per-IP tracking.
- Default: 200 requests per 60 seconds
- Configurable per route or globally
- Refill rate prevents burst abuse

### Honeypot Traps

Fake admin routes deployed to detect scanners and attackers:
- `/wp-admin`, `/wp-login.php` (WordPress)
- `/admin/` (Django)
- `/phpmyadmin/`, `/adminer/` (Database tools)

Three response modes:
| Mode | Behavior |
|------|----------|
| **Ban** | Immediate IP ban (default) |
| **Watch** | Passive intelligence gathering — logs credentials, paths, user agents, timestamps |
| **Blackhole** | Tarpit mode with configurable delay (default 5 seconds per response) |

Credential capture masks passwords in logs (first character + asterisks, never full plaintext).

### Strike System & Integration

- Per-IP strike counter with configurable threshold (default: auto-ban after 5 strikes)
- Watched IP intelligence: first/last seen, captured credentials, visited paths, user agents
- **fail2ban integration**: writes fail2ban-compatible log format for OS-level IP blocking
- **pf integration**: direct kernel table updates via `pfctl` on OpenBSD/FreeBSD

---

## Layer 2: Transport Security (USD)

**Source:** `src/e2e/encrypt.rs`, `src/security/identity.rs`, `src/relay/server.rs`

### End-to-End Encryption

- **Cipher**: XChaCha20-Poly1305 (AEAD — authenticated encryption with associated data)
- **Key Exchange**: Ephemeral ECDH (x25519-dalek) for perfect forward secrecy
- **Key Derivation**: HKDF with domain separation — both public keys included in info field
- **Nonce**: Random per message
- **Signatures**: Optional Ed25519 over header + ciphertext
- **Verification Order**: Signature verified *before* decryption attempt (defense in depth)
- **Tamper Detection**: Poly1305 authentication tag catches any ciphertext or header modification

### Mutual TLS (mTLS)

Three authentication modes:

| Mode | Behavior |
|------|----------|
| **None** | No certificate validation |
| **DirectMtls** | Direct TLS certificate verification |
| **Proxy** | Validates `x-ssl-client-verify: SUCCESS` header from reverse proxy |

X509 certificate parsing extracts:
- CN → node_id
- OU → role
- URI SANs → permissions (`usd://permission/<codename>`), tags (`usd://tag/<tag>`), roles (`usd://role/<role>`)
- SHA-256 fingerprint computation for identity tracking

### Timing Attack Prevention

```rust
pub fn constant_time_eq(a: &[u8], b: &[u8]) -> bool {
    use subtle::ConstantTimeEq;
    a.ct_eq(b).into()
}
```

All cryptographic comparisons use the `subtle` crate for constant-time equality checks.

---

## Layer 3: Message & Protocol Validation (USD)

**Source:** `src/relay/server.rs`, `src/relay/file_transfer.rs`, `src/wasm/mod.rs`, `src/security/mod.rs`

### WebSocket Message Validation

- **Hard size cap**: 4MB per message, enforced before JSON deserialization
- **Strict deserialization**: `serde_json::from_str::<Envelope>()` — only known message types accepted
- **Node ID validation**: 1-64 characters, restricted to alphanumeric + `.`, `_`, `-`

### File Transfer Integrity

| Check | Mechanism |
|-------|-----------|
| Per-chunk integrity | CRC32 checksum verification |
| Sequence validation | Bounds checking, rejects out-of-order chunks |
| Whole-file integrity | SHA-256 hash verification on completion |
| Atomic placement | Write to temp file → verify → `rename()` |
| Path safety | `Path::file_name()` extracts basename only |

### WASM Module Protocol

- Strict prefix matching for protocol commands (`EXEC|`, `FILE_WRITE_ATOMIC|`, etc.)
- Unrecognized lines fall back to `Info()` variant (safe default, not error)
- Custom JSON parser handles escape sequences (`\"`, `\\`, `\n`, `\t`, `\r`)
- Base64 content extracted but not automatically decoded — explicit decode with error handling
- Octal mode validation via `u32::from_str_radix(s, 8)` for file permissions

### Path Sanitization

```rust
fn sanitize_path(path: &str) -> PathValidation
```

- Rejects any path containing `..`
- Hardcoded blocklist of sensitive paths:
  - `/etc/shadow`, `/etc/passwd`, `/etc/spwd.db`, `/etc/pwd.db`
  - `/root/.ssh`, `/var/lib/usd`
  - `/etc/ssh/ssh_host_*`
  - `.ssh/` and `.gnupg/` patterns in home directories
- Optional allowlist — if configured, only listed base paths are permitted
- Returns typed enum: `PathValidation::Allowed` or `PathValidation::Blocked(String)`

### Command Execution

- **Default-deny model**: empty allowlist rejects all commands
- Extracts program basename to prevent path manipulation (`/bin/../evil`)
- Custom shell parser handles single/double quoting and escape sequences
- Rejects unterminated quotes

### Log Injection Prevention

```rust
pub fn sanitize_for_log(s: &str, max_len: usize) -> String
```

- Strips newlines (`\n`) and carriage returns (`\r`)
- Replaces tabs with spaces
- Replaces all control characters with `?`
- Truncates to configurable max length

---

## Layer 4: Authentication & Session Management (rust-primitives)

**Crate:** `primitives-auth` (`crates/identity/auth/`)

### Password Handling

| Aspect | Implementation |
|--------|----------------|
| Hashing | Argon2id |
| Minimum length | 8 characters |
| Empty rejection | Explicit check before hashing |
| Verification | Constant-time comparison |
| Storage | Only hash stored, never plaintext |

### Session Tokens

- 32 bytes (256 bits) cryptographically random
- Base64-URL-safe encoding
- Default 24-hour expiry, configurable
- Validation checks both expiry AND revocation status
- Tracks IP address and user agent per session

### API Keys

- `sk_` prefix for identification
- SHA-256 hashed — never stores plaintext
- Scoped permissions per key
- Optional per-key expiration
- Independent revocation
- `last_used_at` tracking

### Login Attempt Tracking

- Records both successful and failed attempts
- Captures: email, password *length* (never content), IP, user agent
- Links to user ID on success

### Impersonation Controls

- Restricted to staff/admin roles only
- Prevents impersonation of protected roles (other staff/admin)
- Validates both initiator and target roles
- Configurable protected role list

---

## Layer 5: Request Validation (Axum API Layer)

**Sources:** `primitives-api`, `primitives-dto`, axum-patterns, support, scuba-fill-station

### Custom Extractors

- `json_body` extractor returns `ApiError` on parse failure (not framework default)
- `parse_query` with typed error handling
- Custom rejection handlers for `JsonRejection`, `QueryRejection`, `PathRejection`
- Generic error messages — no internal details leaked to client

### Declarative Validation (validator crate)

```rust
#[derive(Validate)]
pub struct CreateTicketRequest {
    #[validate(length(min = 1, max = 255))]
    pub subject: String,
    #[validate(length(max = 10_000))]
    pub description: String,
    #[validate(length(max = 100))]
    pub category: Option<String>,
}
```

Applied to all request DTOs at the deserialization boundary.

### Enum-Based Input Restriction

Priority, status, and transition fields use Rust enums with `FromStr` implementations:

```rust
impl FromStr for TicketStatus {
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s.to_lowercase().as_str() {
            "open" => Ok(Self::Open),
            "in_progress" => Ok(Self::InProgress),
            // ...
            _ => Err(format!("invalid status: {}", s)),
        }
    }
}
```

Free-form strings are never accepted where a finite set of values is expected.

### Tenant Isolation

- `X-Tenant-ID` header extraction with strict UUID format validation
- Per-tenant database connection pool injection into request extensions
- Returns 400 (missing), 401 (invalid format), or 403 (unauthorized) as appropriate

### User Enumeration Prevention

Authentication endpoints return generic error messages:

```
"Invalid email or password"
```

Never reveals whether the email exists or the password was wrong.

### Pagination Limits

All list endpoints enforce default limits (50-100 rows) to prevent resource exhaustion queries.

---

## Layer 6: Type System Enforcement (Compile Time)

### Newtype Pattern

```rust
pub struct PartyId(Uuid);
pub struct UserId(Uuid);
pub struct TenantId(Uuid);
```

- Prevents mixing IDs across entity types at compile time
- A `PartyId` cannot be passed where a `UserId` is expected
- Eliminates ID confusion attacks entirely

### Serde Deserialization

All external input passes through serde deserialization, which:
- Rejects unknown fields (when configured)
- Enforces type constraints (strings, numbers, UUIDs, dates)
- Fails on malformed JSON before any business logic executes

### Result-Based Error Handling

- Custom error types with `thiserror` — no panics on invalid input
- Every fallible operation returns `Result<T, E>` with typed error variants
- `forbid(unsafe_code)` across the entire workspace

---

## Layer 7: Database Constraints (PostgreSQL)

### Compile-Time Checked Queries (SQLx)

```rust
let user = sqlx::query_as!(
    User,
    "SELECT * FROM users WHERE email = $1",
    email
)
.fetch_optional(&pool)
.await?;
```

- SQL queries checked against the database schema at compile time
- Parameterized by default — `$1` bind parameters, never string interpolation
- Type mismatches between Rust code and database columns are compile errors

### Database-Level Invariants

| Constraint | Purpose |
|------------|---------|
| `NOT NULL` | Prevents missing required fields |
| `UNIQUE` | Prevents duplicate emails, slugs, keys |
| `CHECK` | Enforces value ranges and formats |
| `FOREIGN KEY` | Prevents orphaned references |
| `deleted_at IS NULL` | Soft delete filtering in views/queries |

Invariants are enforced in PostgreSQL even if application code has a gap. The database is the final authority.

---

## File & Storage Security

### rust-primitives Storage (`crates/infrastructure/storage/`)

| Control | Implementation |
|---------|----------------|
| Path traversal | Rejects paths containing `..` |
| File size | 10MB default limit, configurable |
| Integrity | SHA-256 checksum on all stored files |
| Organization | Date-based directory structure (`YYYY/MM/DD/UUID.ext`) |

### USD File Transfers (`src/relay/file_transfer.rs`)

| Control | Implementation |
|---------|----------------|
| Per-chunk integrity | CRC32 verification |
| Whole-file integrity | SHA-256 verification |
| Atomic writes | Temp file → verify → `rename()` |
| Path safety | `file_name()` basename extraction |
| Compression | Gzip with deduplication |

### Plumber HTML Archives (`src/storage/html_archive.rs`)

| Control | Implementation |
|---------|----------------|
| Deduplication | SHA-256 content-addressed storage |
| Atomic writes | Temp file + `fsync` + `rename` |
| Path safety | SHA-256 prefix-based directory structure |
| Compression | Gzip archival |

---

## Attack Vector Coverage Summary

| Attack | Layer | Mechanism |
|--------|-------|-----------|
| SQL Injection | WAF + SQLx | 18 regex patterns + compile-time parameterized queries |
| XSS | WAF | 15+ detection patterns for scripts, handlers, protocols |
| Path Traversal | WAF + USD security + Storage | Regex detection + blocklist + `..` rejection |
| Brute Force | WAF | Token bucket rate limiting + strike counter + IP banning |
| Credential Stuffing | WAF | Honeypot traps with ban/watch/blackhole modes |
| Data Exfiltration | WAF | SSN/credit card/API key detection in responses |
| Session Hijacking | Auth | IP/user-agent tracking, revocation, 24hr expiry |
| Timing Attacks | USD | `subtle::ConstantTimeEq` for all crypto comparisons |
| Message Tampering | USD | Poly1305 auth tags + Ed25519 signatures |
| Log Injection | USD | Control character stripping, length truncation |
| Command Injection | USD | Default-deny allowlist, basename extraction |
| User Enumeration | API | Generic "Invalid email or password" responses |
| Resource Exhaustion | API + WAF | Pagination limits + rate limiting + 4MB message cap |
| ID Confusion | Type System | Newtype wrappers prevent cross-entity ID mixing |
| Schema Drift | SQLx | Compile-time query checking against live schema |

---

## Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Rust (zero unsafe code) |
| Web Framework | Axum 0.7 with Tower middleware |
| Database | PostgreSQL with SQLx 0.8 (compile-time checked) |
| Password Hashing | Argon2id |
| Encryption | XChaCha20-Poly1305, x25519-dalek ECDH |
| Signatures | Ed25519 |
| Key Derivation | HKDF |
| Timing Safety | subtle crate (constant-time) |
| Validation | validator crate, custom extractors, serde |
| Error Handling | thiserror 2.x |
| Certificate Parsing | x509-parser |
| OS Integration | fail2ban, pf (pfctl) |
