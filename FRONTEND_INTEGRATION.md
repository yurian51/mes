# Frontend Integration & Setup Guide

## Overview
The frontend has been fully integrated with the backend API layer with proper authentication, error handling, and loading states.

## Files Created

### 1. API Service Layer (`apps/frontend/src/services/api.ts`)
- Centralized Axios instance with request/response interceptors
- Automatic injection of `Authorization: Bearer <token>` header
- Automatic injection of `x-tenant-id: <tenant-id>` header
- Error handling with 401 redirect to login
- Methods: `get()`, `post()`, `put()`, `patch()`, `delete()`

**Usage in Components:**
```typescript
import { apiService } from '@/services/api';

// Fetch data
const data = await apiService.get('/students');

// Create data
const result = await apiService.post('/students', studentData);

// Update data
const updated = await apiService.put(`/students/${id}`, updates);

// Delete data
await apiService.delete(`/students/${id}`);
```

### 2. Custom Hooks (`apps/frontend/src/hooks/useApi.ts`)
Complete set of hooks for all modules:

#### Generic Hook
- `useApi<T>(url, options)` - Generic data fetching with loading/error states

#### Marks Module
- `useMarks(classId?)` - Fetch marks for a class
- `useCreateMark()` - Create/update marks with error handling

#### Students Module  
- `useStudents(tenantId?)` - Fetch all students
- `useStudent(studentId)` - Fetch single student
- `useCreateStudent()` - CRUD operations for students

#### Teachers Module
- `useTeachers()` - Fetch all teachers
- `useTeacher(teacherId)` - Fetch single teacher
- `useCreateTeacher()` - CRUD operations for teachers

#### Finance Module
- `useFinanceData()` - Get dashboard statistics
- `useOutstandingFees(classId?)` - Fetch outstanding fees
- `usePayments(studentId?)` - Fetch payment history
- `useRecordPayment()` - Record new payment

#### Exams & Results
- `useExams(classId?)` - Fetch exams
- `useResults(examId)` - Get exam results
- `useStudentResults(studentId)` - Get student's all results

#### Attendance
- `useAttendance(classId, date?)` - Get attendance records
- `useStudentAttendance(studentId)` - Get student's attendance
- `useRecordAttendance()` - Record attendance

#### Notifications
- `useNotifications()` - Fetch notifications
- `useSendNotification()` - Send and manage notifications

#### Parents Module
- `useParentStudents()` - Get parent's children
- `useParentBalance(studentId)` - Get student's fee balance

#### Library Module
- `useLibraryResources()` - Get library inventory
- `useBorrowResource()` - Borrow and return resources

**Hook Usage Pattern:**
```typescript
'use client';

import { useStudents, useToast } from '@/hooks';

export default function StudentsList() {
  const { data, loading, error } = useStudents();
  const { success, error: errorToast } = useToastWithError();

  if (loading) return <SkeletonTable />;
  if (error) return <div className="text-red-600">{error}</div>;

  return (
    <table className="w-full">
      <tbody>
        {data?.map(student => (
          <tr key={student.id}>
            <td>{student.firstName}</td>
            <td>{student.email}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### 3. Enhanced Authentication Context (`apps/frontend/src/context/AuthContext.tsx`)
- Integrates with API service for automatic header injection
- Persists token and user to localStorage
- Auto-recovery on app reload
- Logout clears all auth state

**Usage:**
```typescript
import { useAuth } from '@/context/AuthContext';

function MyComponent() {
  const { user, token, login, logout, isLoading } = useAuth();

  if (isLoading) return <div>Loading...</div>;
  if (!user) return <div>Not authenticated</div>;

  return (
    <>
      <p>Welcome {user.firstName}</p>
      <p>Tenant: {user.tenantId}</p>
      <button onClick={logout}>Logout</button>
    </>
  );
}
```

### 4. Error Boundary (`apps/frontend/src/components/ErrorBoundary.tsx`)
- Catches React component errors
- Displays error message with reload button
- Prevents white screen of death

**Usage:**
```typescript
import { ErrorBoundary } from '@/components/ErrorBoundary';

export default function Layout() {
  return (
    <ErrorBoundary>
      <YourContent />
    </ErrorBoundary>
  );
}
```

### 5. Toast System (`apps/frontend/src/components/Toast.tsx`)
- Global notification system
- Types: success, error, warning, info
- Auto-dismiss with configurable duration
- Manual dismiss button

**Usage:**
```typescript
import { useToast, useToastWithError } from '@/components/Toast';

function MyComponent() {
  const { addToast } = useToast();
  const toast = useToastWithError();

  async function handleSave() {
    try {
      await apiService.post('/students', data);
      toast.success('Student saved successfully');
    } catch (error) {
      toast.error(error); // Auto-extracts error message
    }
  }

  return <button onClick={handleSave}>Save</button>;
}
```

### 6. Skeleton Loaders (`apps/frontend/src/components/Skeleton.tsx`)
- Reusable skeleton components for loading states
- Types: Skeleton, SkeletonRow, SkeletonCard, SkeletonTable

**Usage:**
```typescript
import { SkeletonTable } from '@/components/Skeleton';
import { useStudents } from '@/hooks/useApi';

function StudentsPage() {
  const { data, loading } = useStudents();

  if (loading) return <SkeletonTable rows={5} columns={4} />;

  return <StudentsList data={data} />;
}
```

### 7. Providers Setup (`apps/frontend/src/app/providers.tsx`)
- Wraps entire app with necessary providers
- ErrorBoundary → AuthProvider → ToastProvider nesting

**Already integrated in** `apps/frontend/src/app/layout.tsx`

### 8. Environment Configuration (`apps/frontend/.env.local`)
```
NEXT_PUBLIC_API_URL=http://localhost:3001
```

Change `NEXT_PUBLIC_API_URL` based on deployment environment:
- **Development**: `http://localhost:3001`
- **Production**: `https://api.yourdomain.com`

## Setup Instructions

### Step 1: Install Dependencies
```bash
npm install
```

### Step 2: Ensure Database is Running
```bash
# PostgreSQL should be running on localhost:5432
# Or update DATABASE_URL in .env.development
```

### Step 3: Run Prisma Migration
```bash
npx prisma migrate dev --name add_advanced_features
```

Or push schema directly:
```bash
npx prisma db push
```

### Step 4: Start Backend Server
From `apps/backend/`:
```bash
npm run start:dev
```

Should start on `http://localhost:3001`

### Step 5: Start Frontend Server
From `apps/frontend/`:
```bash
npm run dev
```

Should start on `http://localhost:3000`

### Step 6: Test Authentication Flow
1. Navigate to `http://localhost:3000/login`
2. Enter credentials
3. On successful login, token and user are automatically set in API service
4. All subsequent API calls include auth headers automatically

## Complete Example: Students Management Page

```typescript
'use client';

import { useState } from 'react';
import { useStudents, useCreateStudent } from '@/hooks/useApi';
import { useToastWithError } from '@/components/Toast';
import { SkeletonTable } from '@/components/Skeleton';
import { DataTable } from '@/components/DataTable';

export default function StudentsPage() {
  const { data, loading, error, refetch } = useStudents();
  const { create } = useCreateStudent();
  const toast = useToastWithError();
  const [isOpen, setIsOpen] = useState(false);

  const handleAddStudent = async (formData: any) => {
    try {
      await create(formData);
      toast.success('Student added successfully');
      setIsOpen(false);
      refetch(); // Refresh the list
    } catch (error) {
      toast.error(error);
    }
  };

  if (loading) return <SkeletonTable />;
  if (error) return <div className="text-red-600">Error: {error}</div>;

  return (
    <div className="p-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Students</h1>
        <button
          onClick={() => setIsOpen(true)}
          className="bg-blue-600 text-white px-4 py-2 rounded-lg"
        >
          Add Student
        </button>
      </div>

      <DataTable columns={columns} data={data || []} />

      {isOpen && (
        <StudentForm
          onSubmit={handleAddStudent}
          onClose={() => setIsOpen(false)}
        />
      )}
    </div>
  );
}
```

## Available API Endpoints

### Marks
- `GET /marks` - List marks
- `POST /marks` - Create mark
- `PUT /marks/:id` - Update mark
- `DELETE /marks/:id` - Delete mark

### Students
- `GET /students` - List students
- `POST /students` - Create student
- `PUT /students/:id` - Update student
- `DELETE /students/:id` - Delete student

### Teachers
- `GET /teachers` - List teachers
- `POST /teachers` - Create teacher
- `PUT /teachers/:id` - Update teacher
- `DELETE /teachers/:id` - Delete teacher

### Finance
- `GET /finance/dashboard` - Get statistics
- `GET /finance/outstanding-fees` - Get outstanding fees
- `GET /finance/payments` - Get payment history
- `POST /finance/payments` - Record payment

### Exams & Results
- `GET /exams` - List exams
- `GET /exams/:id/results` - Get exam results
- `GET /exams/student/:studentId/results` - Get student results

### Attendance
- `GET /attendance` - Get attendance records
- `GET /attendance/student/:studentId` - Get student attendance
- `POST /attendance` - Record attendance

### Notifications
- `GET /notifications` - Get notifications
- `POST /notifications` - Send notification
- `PUT /notifications/:id/read` - Mark as read

### Parents
- `GET /parents/students` - Get parent's students
- `GET /parents/students/:studentId/balance` - Get student balance

### Library
- `GET /library/resources` - List resources
- `POST /library/borrow` - Borrow resource
- `POST /library/return/:borrowId` - Return resource

## Authentication Details

### Login Flow
1. Call `POST /auth/login` with credentials
2. Receive `{ token, user }` in response
3. Call `login(token, user)` from `useAuth()`
4. API service automatically injects:
   - `Authorization: Bearer <token>`
   - `x-tenant-id: <user.tenantId>`

### Headers Requirement
ALL requests must include:
```
Authorization: Bearer <jwt_token>
x-tenant-id: <tenant-id>
```

The API service handles this automatically when authenticated.

### Token Expiration
- Tokens set to expire in `JWT_EXPIRES_IN` (.env)
- On 401 error, user is redirected to `/login`
- Token stored in localStorage (cleared on logout)

## Troubleshooting

### "Cannot find module '@/services/api'"
- Ensure tsconfig paths are configured correctly
- Check that `@/` maps to `src/`

### API Service Not Setting Headers
- Verify `login()` is called with user data
- Check that `user.tenantId` exists
- Open DevTools → Network → check request headers

### 401 Unauthorized Responses
- Token may be expired
- Check localStorage for token
- Try logging in again

### CORS Errors
- Backend should have CORS enabled
- Check `NEXT_PUBLIC_API_URL` is correct
- Verify backend is running

## Next Steps

1. **Test API endpoints** with Postman or Thunder Client
   - Set `Authorization` header with valid token
   - Set `x-tenant-id` header with tenant UUID

2. **Connect dashboard pages** to use hooks
   - Replace mock data with API calls
   - Add error handling and loading states

3. **Implement forms** for data submission
   - Use `useCreateStudent()`, `useCreateTeacher()` etc.
   - Show success/error toasts on completion

4. **Add real-time features** (optional)
   - WebSocket for notifications
   - Redis pub/sub for live updates

## Support

For issues or questions, check:
- API response status codes
- Browser console for errors
- Network tab in DevTools
- Backend logs for detailed errors
