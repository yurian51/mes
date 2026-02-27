# MES API Documentation

## Authentication
- `POST /auth/login`: Authenticate and receive JWT. Requires `x-tenant-id` header.
- `POST /auth/register-school`: Central SaaS registration.

## Tenants
- `GET /tenants/me`: Fetch current school profile.
- `PATCH /tenants/:id`: Update school settings (Logo, Grading).

## Academic
- `GET /students`: List students with pagination.
- `POST /marks/bulk-upload`: Bulk score entry.
- `POST /results/compute`: Trigger computation engine for a term/exam.

## Finance
- `POST /fees/structure`: Define fee categories.
- `POST /invoices/generate`: Automated billing.
- `POST /payments/webhook/:provider`: Bank/Mobile money reconciliation.

## Security
All endpoints (except login) require:
1. `Authorization: Bearer <token>`
2. `x-tenant-id: <uuid>`
