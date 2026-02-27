# MES System Architecture

## Overview
MES is a professional multi-tenant School Management SaaS built with a modern monorepo architecture. 

## Tech Stack
- **Frontend**: Next.js 14 (App Router), Tailwind CSS, Framer Motion.
- **Backend**: NestJS (Modular Architecture), Prisma ORM.
- **Database**: PostgreSQL with performance-tuned indexing.
- **Cache**: Redis for session and computation caching.
- **Containerization**: Docker Multi-stage production builds.

## Monorepo Structure
- `apps/backend`: NestJS API.
- `apps/frontend`: Next.js Client.
- `packages/database`: Shared Prisma schema and client.
- `packages/ui`: Shared Enterprise Design System.
- `packages/config`: Shared TSConfig and linting rules.

## Core Design Patterns
1. **Multi-Tenancy**: Logical isolation using `tenant_id` on all tables.
2. **Soft Delete**: All core records use `deletedAt` for data safety.
3. **Audit Trails**: Global logging of all mutating actions.
4. **Result computation pipeline**: Deterministic logic for grading and ranking.
