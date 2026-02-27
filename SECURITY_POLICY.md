# MES Security Policy

## Core Security Pillars

### 1. Multi-Tenant Isolation
- Every query is strictly scoped using the `tenant_id` index.
- Middleware ensures that `x-tenant-id` header is verified against the authenticated user's permissions.

### 2. Data Integrity
- **Audit Logging**: Every mutating action (POST/PATCH/DELETE) is captured with `oldValue` and `newValue`.
- **Soft Deletes**: Accidental data deletion is prevented via a global `deletedAt` filter.

### 3. API Hardening
- **Helmet**: Secure HTTP headers enabled.
- **Rate Limiting**: Brute-force protection on authentication endpoints.
- **Input Validation**: `class-validator` enforces strict types on all incoming payloads.

### 4. Financial Security
- Webhook signatures are verified before processing bank/mobile money payments.
- Idempotency keys/Reference checks prevent duplicate payment records.
