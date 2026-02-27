# MES Deployment Guide

This guide covers the production deployment of the MES School Management SaaS.

## Prerequisites
- Docker & Docker Compose
- Node.js 18+
- PostgreSQL (if not using Docker for DB)
- Domain with SSL support

## Step 1: Environment Configuration
Copy `.env.example` to `.env` and update the production secrets:
- `JWT_SECRET`: Generate a strong random string.
- `DATABASE_URL`: Your production PostgreSQL connection string.
- `REDIS_HOST`: Your production Redis host.

## Step 2: Infrastructure Setup
Run the supporting services using Docker Compose:
```bash
docker-compose up -d postgres redis
```

## Step 3: Database Migration
Generate the Prisma client and push the schema to the database:
```bash
npm install
npm run build -w @mes/database
npx prisma migrate deploy --schema packages/database/prisma/schema.prisma
```

## Step 4: Build & Run Apps
### Backend
```bash
cd apps/backend
npm run build
npm run start:prod
```

### Frontend
```bash
cd apps/frontend
npm run build
npm run start
```

## Step 5: Post-Deployment
1. Seed the `master_subjects` table with Tanzanian subjects.
2. Create the first Super Admin tenant.
3. Configure your reverse proxy (Nginx/Traefik) to handle subdomains if using subdomain-based tenant isolation.
