# bank-core

Backend for a mobile banking app. One core, two money adapters:
`SimulatedBank` (fake money, for building/testing) and `PartnerBank` (real sponsor-bank API, later).
Simulated and real environments must never share a database, keys or credentials.

## Status
Phase 2, step 1: skeleton, schema, security baseline, CI. NOT production-ready.
Not yet built: device registration, token issuing/validation hardening, payments, fraud rules, admin tools.

## First run
    gradle wrapper --gradle-version 8.10.2     # one-time, generates ./gradlew
    ./gradlew test                              # needs Docker for LedgerSchemaIT

Required environment variables (no defaults, by design):
`DB_URL`, `DB_USER`, `DB_PASSWORD`, `JWT_JWK_SET_URI`; optional `BANK_ENV` (simulated|partner).

## Database rules enforced by Postgres itself
- Ledger transactions must sum to zero per currency (checked at commit)
- `ledger_entries`, `ledger_transactions`, `audit_log` are append-only (UPDATE/DELETE blocked)
- Customer accounts must have an owner; devices carry a cooling-off `activates_at`

## Production DB roles (do before any real deployment)
- Migration role: owns schema, used only by CI/CD
- App role: SELECT/INSERT on ledger and audit tables only, no DDL, no UPDATE/DELETE there

## Known TODOs for the next steps
- Validate token issuer and audience (jwk-set-uri alone does not check them)
- Device-key signature verification and replay protection on payments
- Field-level encryption for the PII profile table
- Hash-chain computation for audit_log rows
- Concurrency control (row locks) so balances cannot be overdrawn by parallel payments
