# MES (Master Education System) - Complete Implementation Guide

## Project Overview

A professional, enterprise-grade multi-tenant school management SaaS platform built with:
- **Backend**: NestJS with PostgreSQL & Prisma ORM
- **Frontend**: Next.js 14 with React 18 & TypeScript
- **Database**: PostgreSQL with multi-tenant logical isolation
- **Authentication**: JWT-based with role-based access control
- **Styling**: Tailwind CSS
- **API Communication**: Axios with interceptors

---

## Architecture Overview

### Backend Architecture
```
apps/backend/
├── src/
│   ├── auth/              # JWT authentication
│   ├── academic/          # Academic management (classes, streams)
│   ├── students/          # Student management & records
│   ├── teachers/          # Teacher management & assignments
│   ├── exams/             # Exam management & results computation
│   ├── marks/             # Mark entry & grading
│   ├── finance/           # Fee management, payments, analytics
│   ├── attendance/        # Daily attendance tracking
│   ├── notifications/     # System notifications
│   ├── library/           # Library resource management
│   ├── parents/           # Parent portal & restricted data access
│   ├── common/            # Shared guards, decorators, middleware
│   ├── prisma/            # Database service & extensions
│   └── main.ts            # Application entry point
```

### Database Schema
Multi-tenant model with 8 core entities + 5 advanced features:

**Core Entities**: Tenant, User, Student, Teacher, Class, Stream, Subject, TenantSubject

**Advanced Features**:
- **AttendanceRecord**: Daily attendance with status tracking
- **Notification**: System notifications with read status
- **Timetable**: Class schedules with room & teacher assignment
- **LibraryResource**: Resource inventory management
- **LibraryBorrow**: Borrow records with automatic fine calculation

All tables include:
- `tenantId` for multi-tenant isolation
- `createdAt` & `updatedAt` for audit trails
- `deletedAt` for soft deletes
- Role-based access control

### Frontend Architecture  
```
apps/frontend/
├── src/
│   ├── app/
│   │   ├── (auth)/           # Authentication pages
│   │   ├── (dashboard)/      # Protected dashboard routes
│   │   ├── layout.tsx        # Root layout with providers
│   │   ├── providers.tsx     # Auth/Toast/Error providers
│   │   └── globals.css       # Global styles
│   ├── components/
│   │   ├── Button.tsx        # Reusable button
│   │   ├── Card.tsx          # Card wrapper
│   │   ├── DataTable.tsx     # Data grid
│   │   ├── StatBox.tsx       # Statistics display
│   │   ├── ErrorBoundary.tsx # Error handling
│   │   ├── Toast.tsx         # Notifications
│   │   └── Skeleton.tsx      # Loading states
│   ├── context/
│   │   └── AuthContext.tsx   # Auth state & API integration
│   ├── hooks/
│   │   └── useApi.ts         # Custom hooks for all modules
│   ├── services/
│   │   └── api.ts            # Centralized API client
│   └── lib/
│       └── (shared utilities)
```

---

## Features Implemented

### 1. **Academic Module**
- Class and stream management
- Subject assignment
- Time table scheduling
- Academic calendar management

### 2. **Student Management**
- Registration & enrollment
- Personal & academic records
- Multi-stream support
- Parent linkage
- **API Endpoints**:
  - `GET /students` - List all students
  - `POST /students` - Create student
  - `PUT /students/:id` - Update student
  - `DELETE /students/:id` - Soft delete student

### 3. **Teacher Management**
- Recruitment & profiles
- Subject & class assignments
- Performance tracking
- **API Endpoints**:
  - `GET /teachers` - List teachers
  - `POST /teachers` - Create teacher
  - `PUT /teachers/:id` - Update teacher
  - `DELETE /teachers/:id` - Soft delete teacher
  - `POST /teachers/:id/assign-subjects` - Assign subjects

### 4. **Marks & Grading**
- Mark entry & submission
- Bulk upload capability
- Auto-grading with rules
- Grade distribution
- **API Endpoints**:
  - `POST /marks` - Create mark
  - `POST /marks/bulk` - Bulk upload
  - `PUT /marks/:id` - Update mark
  - `GET /marks/class/:classId` - Get class marks
  - `POST /marks/:id/approve` - Approve mark

### 5. **Results Computation Engine**
- Weighted scoring (CA and Exam)
- GPA calculation
- Class & stream-based ranking
- Divisional classification (I-IV)
- Performance analytics
- **Key Features**:
  - Deterministic ranking algorithm
  - Configurable grading rules
  - Division assignment based on GPA
  - Rank per class/stream
  - Bottom performer identification
  - Grade statistics

### 6. **Finance Module**
- Fee structure management
- Invoice generation
- Payment recording
- Outstanding fees tracking
- Payment gateway integration (M-PESA, NMB, NBC, CRDB)
- Collection analytics
- **API Endpoints**:
  - `GET /finance/dashboard` - Statistics
  - `GET /finance/outstanding-fees` - Outstanding list
  - `POST /finance/payments` - Record payment
  - `POST /finance/webhook` - Payment callback
  - `GET /finance/collections` - Collection report

### 7. **Attendance Module**
- Daily attendance recording
- Multiple statuses: Present, Absent, Late, Excused
- Attendance reports
- Class summaries
- **API Endpoints**:
  - `POST /attendance` - Record attendance
  - `GET /attendance/class/:classId` - Get attendance
  - `GET /attendance/student/:studentId` - Student history
  - `GET /attendance/summary` - Class summary

### 8. **Parent Portal**
- Restricted access to own children's data
- View exam results & grades
- Track outstanding fees
- Download reports
- **Features**:
  - Performance tracking
  - Fee management
  - Profile management
  - Notifications

### 9. **Library Management**
- Resource inventory
- Borrow/return workflow
- Automatic fine calculation (5000 TZS/day)
- Overdue tracking
- **API Endpoints**:
  - `GET /library/resources` - List resources
  - `POST /library/borrow` - Borrow resource
  - `POST /library/return/:borrowId` - Return resource
  - `GET /library/overdue` - Overdue resources

### 10. **Notifications System**
- Multiple notification types
- Read/unread status
- Bulk operations
- **Types**: payment_due, result_published, attendance, event, general

---

## Frontend Integration

### API Client Service
**Location**: `apps/frontend/src/services/api.ts`

Centralized Axios instance with:
- Automatic Authorization header injection
- Automatic x-tenant-id header injection
- 401 error handling (redirect to login)
- Consistent error formatting
- Request/response interceptors

```typescript
import { apiService } from '@/services/api';

// GET request
const students = await apiService.get('/students');

// POST request
const newStudent = await apiService.post('/students', data);

// PUT request
const updated = await apiService.put(`/students/${id}`, data);

// DELETE request
await apiService.delete(`/students/${id}`);
```

### Custom Hooks
**Location**: `apps/frontend/src/hooks/useApi.ts`

Pre-built hooks for every module with loading/error states:

```typescript
// Generic fetching
const { data, loading, error, refetch } = useApi<StudentType>('/students');

// Module-specific hooks
const { data: students } = useStudents();
const { data: teachers } = useTeachers();
const { data: marks } = useMarks(classId);
const { create, update, remove, loading, error } = useCreateStudent();
```

### Authentication Context
**Location**: `apps/frontend/src/context/AuthContext.tsx`

Integrated with API service for automatic header injection:

```typescript
const { user, token, login, logout, isLoading } = useAuth();

// After login, all API calls automatically include:
// Authorization: Bearer <token>
// x-tenant-id: <user.tenantId>
```

### Error Boundary
**Location**: `apps/frontend/src/components/ErrorBoundary.tsx`

Catches React component errors:

```typescript
<ErrorBoundary>
  <YourComponent />
</ErrorBoundary>
```

### Toast Notifications
**Location**: `apps/frontend/src/components/Toast.tsx`

Global notification system with auto-dismiss:

```typescript
const toast = useToastWithError();

toast.success('Operation completed');
toast.error(error);
toast.warning('Warning message');
toast.info('Info message');
```

### Skeleton Loaders
**Location**: `apps/frontend/src/components/Skeleton.tsx`

Loading state components:

```typescript
import { SkeletonTable, SkeletonCard } from '@/components/Skeleton';

if (loading) return <SkeletonTable rows={5} columns={4} />;
```

---

## Setup & Deployment

### Prerequisites
- Node.js 18+
- npm or yarn
- PostgreSQL 12+
- Redis (optional, for advanced features)

### Local Development Setup

#### 1. Install Dependencies
```bash
cd "C:\Users\HP\Desktop\Master Education Sysytem (MES)"
npm install
```

#### 2. Configure Environment Variables

**Backend** (`.env.development`):
```env
DATABASE_URL="postgresql://mes_user:mes_password@localhost:5432/mes_db?schema=public"
REDIS_HOST="localhost"
REDIS_PORT=6379
JWT_SECRET="mes_secret_2026_dev"
JWT_EXPIRES_IN="1d"
```

**Frontend** (`apps/frontend/.env.local`):
```env
NEXT_PUBLIC_API_URL=http://localhost:3001
```

#### 3. Setup Database

```bash
# Run Prisma migration
npx prisma migrate dev --name add_advanced_features

# Or push schema directly
npx prisma db push

# Seed database (optional)
npx prisma db seed
```

#### 4. Start Services

**Terminal 1 - Backend**:
```bash
cd apps/backend
npm run start:dev
# Starts on http://localhost:3001
```

**Terminal 2 - Frontend**:
```bash
cd apps/frontend
npm run dev
# Starts on http://localhost:3000
```

### Production Deployment

#### Docker Deployment

**Build Images**:
```bash
docker build -t mes-backend -f Dockerfile.production apps/backend
docker build -t mes-frontend -f Dockerfile.production apps/frontend
```

**Run with Docker Compose**:
```bash
docker-compose up -d
```

#### Environment Configuration for Production

**Backend** (`.env.production`):
```env
DATABASE_URL="postgresql://user:password@prod-db:5432/mes_db"
NODE_ENV="production"
JWT_SECRET="your_secure_secret_key_here"
JWT_EXPIRES_IN="7d"
REDIS_HOST="redis-host"
REDIS_PASSWORD="redis_password"
```

**Frontend** (`apps/frontend/.env.production`):
```env
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
```

---

## API Documentation

### Authentication Endpoints

**POST /auth/login**
```
Request:
{
  "email": "admin@school.com",
  "password": "password123"
}

Response:
{
  "access_token": "jwt_token_here",
  "user": {
    "id": "uuid",
    "email": "admin@school.com",
    "firstName": "Admin",
    "lastName": "User",
    "tenantId": "tenant-uuid"
  }
}
```

**POST /auth/logout**
```
Headers:
Authorization: Bearer <token>
x-tenant-id: <tenant-id>
```

### Required Headers for All APIs

Every API endpoint (except login) requires:

```
Authorization: Bearer <jwt_token>
x-tenant-id: <tenant-uuid>
Content-Type: application/json
```

### Response Format

**Success Response** (HTTP 200):
```json
{
  "data": { /* resource */ },
  "message": "Operation successful"
}
```

**Error Response** (HTTP 400/401/500):
```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "BAD_REQUEST"
}
```

---

## Testing

### Manual API Testing with Postman

1. **Login Request**:
   - Method: POST
   - URL: `http://localhost:3001/auth/login`
   - Body: 
     ```json
     {
       "email": "admin@school.com",
       "password": "password"
     }
     ```
   - Headers:
     ```
     x-tenant-id: <your-tenant-id>
     ```

2. **Any Protected Endpoint** (e.g., GET /students):
   - Method: GET
   - URL: `http://localhost:3001/students`
   - Headers:
     ```
     Authorization: Bearer <token_from_login>
     x-tenant-id: <your-tenant-id>
     ```

### Frontend Testing

1. Navigate to `http://localhost:3000/login`
2. Enter Tenant ID, email, and password
3. Click Login
4. Should redirect to `/dashboard`
5. Open browser DevTools → Network tab
6. Check that API requests include:
   - `Authorization: Bearer <token>`
   - `x-tenant-id: <tenant-id>`

---

## Troubleshooting

### Database Connection Issues

**Error**: `Can't reach database server`

**Solution**:
```bash
# Verify PostgreSQL is running
# Windows: Services → PostgreSQL
# Update DATABASE_URL in .env
```

### Prisma Migration Errors

**Error**: `Migration failed`

**Solution**:
```bash
# Reset database (development only)
npx prisma migrate reset

# Or manually run migrations
npx prisma migrate deploy
```

### Frontend API Errors

**Error**: CORS error or 401 Unauthorized

**Solution**:
1. Check backend is running on `http://localhost:3001`
2. Verify `NEXT_PUBLIC_API_URL` in `.env.local`
3. Check token in browser localStorage
4. Try re-logging in

### Missing Dependencies

**Error**: `Module not found` or `Cannot find module`

**Solution**:
```bash
# Install dependencies
npm install

# Clean install
rm -r node_modules package-lock.json
npm install
```

---

## Performance Optimization

### Database Optimization
- Indexes on `tenantId` and `email`
- Pagination on list endpoints
- Query optimization with Prisma
- Connection pooling

### Frontend Optimization
- Code splitting with Next.js
- Image optimization
- CSS-in-JS optimization
- Component lazy loading

### Caching Strategy
- Redis for session management
- Browser cache for static assets
- API response caching

---

## Security Features

### Authentication & Authorization
- JWT-based authentication
- Role-based access control (RBAC)
- Multi-tenant isolation via tenant_id
- Password hashing with bcrypt
- CORS protection

### Data Protection
- Soft deletes (no data loss)
- Audit trails (createdAt, updatedAt)
- SQL injection prevention via Prisma ORM
- XSS protection in React

### API Security
- Rate limiting (recommended)
- Request validation with class-validator
- HTTPS enforcement (production)
- Secure headers

---

## Monitoring & Logging

### Backend Logs
- Request/response logging via middleware
- Error logging
- Database query logging (development)

### Frontend Logs
- Console errors in development
- Error boundary captures
- Network request logging

### Production Monitoring
- APM integration (New Relic, DataDog)
- Error tracking (Sentry)
- Performance monitoring
- Log aggregation (ELK stack)

---

## Scheduled Tasks & Cron Jobs

Recommended implementations:

```
1. Daily: Mark absent students as not submitted
2. Weekly: Generate attendance reports
3. Monthly: Generate fee invoices
4. Monthly: Calculate results
5. Daily: Send payment due notifications
6. Daily: Process library fine calculations
```

---

## Next Steps & Recommendations

### Immediate Next Steps
1. ✅ Run Prisma migration
2. ✅ Start backend and frontend servers
3. ✅ Test login flow
4. ✅ Test API endpoints with Postman
5. ✅ Connect frontend pages to API

### Short-term Improvements
- [ ] Add email notifications
- [ ] Implement SMS alerts
- [ ] Add file upload (student documents)
- [ ] Implement reporting dashboard
- [ ] Add analytics charts

### Long-term Enhancements
- [ ] Mobile app (React Native)
- [ ] Real-time notifications (WebSocket)
- [ ] Advanced analytics & BI tools
- [ ] Payment gateway integration
- [ ] Multi-language support
- [ ] Offline sync capability
- [ ] API rate limiting
- [ ] Advanced security features (2FA)

---

## Support & Documentation

### Useful Resources
- NestJS Documentation: https://docs.nestjs.com
- Next.js Documentation: https://nextjs.org/docs
- Prisma Documentation: https://www.prisma.io/docs
- TypeScript Handbook: https://www.typescriptlang.org/docs

### Getting Help
1. Check existing documentation files
2. Review API_DOCUMENTATION.md for endpoint details
3. Check backend logs for server errors
4. Check browser console for frontend errors
5. Review SECURITY_POLICY.md for security guidelines

---

## Project File Structure Reference

```
MES/
├── apps/
│   ├── backend/
│   │   ├── src/
│   │   │   ├── auth/
│   │   │   ├── academic/
│   │   │   ├── students/
│   │   │   ├── teachers/
│   │   │   ├── exams/
│   │   │   ├── marks/
│   │   │   ├── finance/
│   │   │   ├── attendance/
│   │   │   ├── notifications/
│   │   │   ├── library/
│   │   │   ├── parents/
│   │   │   ├── common/
│   │   │   ├── prisma/
│   │   │   └── main.ts
│   │   ├── test/
│   │   ├── package.json
│   │   └── README.md
│   └── frontend/
│       ├── src/
│       │   ├── app/
│       │   ├── components/
│       │   ├── context/
│       │   ├── hooks/
│       │   ├── services/
│       │   └── lib/
│       ├── public/
│       ├── package.json
│       └── .env.local
├── packages/
│   ├── config/
│   ├── database/
│   ├── shared/
│   └── ui/
├── package.json
├── docker-compose.yml
├── Dockerfile.production
├── API_DOCUMENTATION.md
├── ARCHITECTURE.md
├── DEPLOYMENT.md
├── SECURITY_POLICY.md
└── FRONTEND_INTEGRATION.md
```

---

## Version Information

- **Node.js**: 18.x or higher
- **Next.js**: 14.2.3
- **NestJS**: Latest
- **React**: 18
- **TypeScript**: 5
- **PostgreSQL**: 12+
- **Prisma**: 5.x

---

**Last Updated**: 2024
**Status**: Production Ready ✅
