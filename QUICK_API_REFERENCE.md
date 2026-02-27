# Quick API Reference Guide

## Quick Start

### 1. Login
```
POST /auth/login
Headers: x-tenant-id: <tenant-uuid>
Body: {
  "email": "admin@school.com",
  "password": "password"
}
Response: { access_token, user }
```

### 2. Store Token
Save `access_token` from login response for all other requests

### 3. All Requests Need
```
Authorization: Bearer <token>
x-tenant-id: <tenant-uuid>
```

---

## Students API

### List Students
```
GET /students
```

### Get Student
```
GET /students/:id
```

### Create Student
```
POST /students
Body: {
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@school.com",
  "phoneNumber": "+255...",
  "dateOfBirth": "2010-05-15",
  "gender": "M",
  "classId": "uuid",
  "admissionNumber": "ADM001"
}
```

### Update Student
```
PUT /students/:id
Body: { /* same as create */ }
```

### Delete Student
```
DELETE /students/:id
```

---

## Teachers API

### List Teachers
```
GET /teachers
```

### Get Teacher
```
GET /teachers/:id
```

### Create Teacher
```
POST /teachers
Body: {
  "firstName": "Jane",
  "lastName": "Smith",
  "email": "jane@school.com",
  "phoneNumber": "+255...",
  "employmentNumber": "EMP001",
  "department": "Mathematics"
}
```

### Update Teacher
```
PUT /teachers/:id
Body: { /* same as create */ }
```

### Assign Subjects
```
POST /teachers/:id/assign-subjects
Body: {
  "classId": "uuid",
  "subjectIds": ["uuid1", "uuid2"]
}
```

### Delete Teacher
```
DELETE /teachers/:id
```

---

## Marks API

### Create Mark
```
POST /marks
Body: {
  "studentId": "uuid",
  "examId": "uuid",
  "caScore": 15,     // Continuous Assessment (out of 20)
  "examScore": 75    // Exam score (out of 100)
}
```

### Get Marks
```
GET /marks?classId=uuid&examId=uuid
```

### Get Class Marks
```
GET /marks/class/:classId
```

### Update Mark
```
PUT /marks/:id
Body: {
  "caScore": 16,
  "examScore": 78,
  "status": "draft"
}
```

### Approve Mark
```
POST /marks/:id/approve
```

### Bulk Upload Marks
```
POST /marks/bulk
Body: {
  "examId": "uuid",
  "marks": [
    { "studentId": "uuid", "caScore": 15, "examScore": 75 },
    { "studentId": "uuid", "caScore": 14, "examScore": 72 }
  ]
}
```

### Delete Mark
```
DELETE /marks/:id
```

---

## Exams & Results API

### Create Exam
```
POST /exams
Body: {
  "name": "Midterm Exam - 2024",
  "description": "First midterm examination",
  "classId": "uuid",
  "startDate": "2024-03-15T09:00:00Z",
  "endDate": "2024-03-25T15:00:00Z",
  "totalMarks": 100,
  "passMarks": 40
}
```

### List Exams
```
GET /exams?classId=uuid
```

### Get Exam Results
```
GET /exams/:id/results
```

### Get Student Results
```
GET /exams/student/:studentId/results
```

### Student Specific Results
```
GET /exams/:examId/results/student/:studentId
```

---

## Finance API

### Get Dashboard
```
GET /finance/dashboard
Response: {
  "totalCollected": 5000000,
  "totalOutstanding": 1500000,
  "collectionRate": "77%",
  "pendingInvoices": 45
}
```

### Create Fee Structure
```
POST /finance/fee-structures
Body: {
  "name": "Tuition 2024",
  "description": "Annual tuition fee",
  "amount": 500000,
  "frequency": "termly",  // termly, yearly, monthly
  "classId": "uuid"
}
```

### List Invoices
```
GET /finance/invoices
```

### Generate Invoices for Class
```
POST /finance/invoices/generate
Body: {
  "classId": "uuid",
  "feeStructureId": "uuid"
}
```

### Get Outstanding Fees
```
GET /finance/outstanding-fees?classId=uuid&studentId=uuid
Response: [
  {
    "studentId": "uuid",
    "studentName": "John Doe",
    "totalOutstanding": 1500000,
    "invoices": [...]
  }
]
```

### Record Payment
```
POST /finance/payments
Body: {
  "invoiceId": "uuid",
  "amount": 500000,
  "paymentMethod": "mpesa",  // mpesa, nmb, nbc, crdb
  "referenceNumber": "TX123456",
  "paymentDate": "2024-03-10T14:30:00Z"
}
```

### Get Payment History
```
GET /finance/payments?studentId=uuid&invoiceId=uuid
```

### Payment Webhook (Payment Provider)
```
POST /finance/webhook
Body: {
  "provider": "mpesa",
  "transactionId": "TX123456",
  "status": "completed",
  "amount": 500000
}
```

### Get Collections Report
```
GET /finance/collections
Query: ?startDate=2024-01-01&endDate=2024-03-31&classId=uuid
```

---

## Attendance API

### Record Attendance
```
POST /attendance
Body: {
  "classId": "uuid",
  "date": "2024-03-15",
  "records": [
    {
      "studentId": "uuid",
      "status": "present"  // present, absent, late, excused
    }
  ]
}
```

### Get Class Attendance
```
GET /attendance?classId=uuid&date=2024-03-15
```

### Get Student Attendance
```
GET /attendance/student/:studentId
Query: ?startDate=2024-01-01&endDate=2024-03-31
```

### Get Class Summary
```
GET /attendance/summary?classId=uuid
Response: {
  "classId": "uuid",
  "totalStudents": 40,
  "averageAttendance": "85%",
  "topAttender": { ... },
  "statusDistribution": {
    "present": 35,
    "absent": 3,
    "late": 2,
    "excused": 0
  }
}
```

---

## Notifications API

### Send Notification
```
POST /notifications
Body: {
  "userId": "uuid",
  "title": "Exam Results Published",
  "message": "Your exam results are now available",
  "type": "result_published",  // payment_due, result_published, attendance, event, general
  "relatedId": "examId"
}
```

### Get Notifications
```
GET /notifications
Query: ?limit=20&page=1
```

### Get Unread
```
GET /notifications/unread
```

### Mark As Read
```
PUT /notifications/:id/read
```

### Mark As Unread
```
PUT /notifications/:id/unread
```

### Mark All As Read
```
POST /notifications/mark-all-read
```

---

## Attendance API

### Create Attendance Record
```
POST /attendance
Body: {
  "classId": "uuid",
  "date": "2024-03-15",
  "records": [
    {
      "studentId": "uuid",
      "status": "present"
    }
  ]
}
```

### List Attendance
```
GET /attendance?classId=uuid&date=2024-03-15
```

### Get Student Attendance
```
GET /attendance/student/:studentId
```

### Get Class Summary
```
GET /attendance/summary?classId=uuid
```

---

## Library API

### Add Resource
```
POST /library/resources
Body: {
  "title": "Mathematics Textbook",
  "author": "John Smith",
  "isbn": "978-3-16-148410-0",
  "category": "Textbook",
  "quantity": 10,
  "location": "Shelf A1"
}
```

### List Resources
```
GET /library/resources
Query: ?category=Textbook&page=1&limit=20
```

### Get Resource
```
GET /library/resources/:id
```

### Update Resource
```
PUT /library/resources/:id
Body: { /* same as create */ }
```

### Borrow Resource
```
POST /library/borrow
Body: {
  "studentId": "uuid",
  "resourceId": "uuid",
  "quantity": 1,
  "dueDate": "2024-04-15"
}
Response: {
  "borrowId": "uuid",
  "borrowDate": "2024-03-15",
  "dueDate": "2024-04-15",
  "status": "active",
  "finePerDay": 5000
}
```

### Return Resource
```
POST /library/return/:borrowId
Body: {
  "returnDate": "2024-03-20"
}
Response: {
  "status": "returned",
  "daysOverdue": 0,
  "fineAmount": 0
}
```

### Get Resource
```
GET /library/resources/:id
```

### Get Overdue Resources
```
GET /library/overdue
Response: [
  {
    "borrowId": "uuid",
    "studentName": "John Doe",
    "resourceTitle": "Math Textbook",
    "dueDate": "2024-03-10",
    "daysOverdue": 5,
    "estimatedFine": 25000
  }
]
```

---

## Parents API

### Get Parent's Students
```
GET /parents/students
Response: [
  {
    "studentId": "uuid",
    "firstName": "John",
    "lastName": "Doe",
    "class": "Form 2",
    "stream": "A"
  }
]
```

### Get Student Balance
```
GET /parents/students/:studentId/balance
Response: {
  "studentId": "uuid",
  "studentName": "John Doe",
  "totalOutstanding": 500000,
  "invoices": [...]
}
```

### Get Student Marks
```
GET /parents/students/:studentId/marks
Response: [
  {
    "subject": "Mathematics",
    "exam": "Midterm 2024",
    "caScore": 15,
    "examScore": 78,
    "total": 93,
    "grade": "A"
  }
]
```

### Get Student Attendance
```
GET /parents/students/:studentId/attendance
```

---

## Academic API

### Create Class
```
POST /academic/classes
Body: {
  "name": "Form 2",
  "description": "Form 2 Classes",
  "classTeacherId": "uuid"
}
```

### Create Stream
```
POST /academic/streams
Body: {
  "name": "A",
  "classId": "uuid",
  "capacity": 45,
  "classTeacherId": "uuid"
}
```

### List Classes
```
GET /academic/classes
```

### List Streams
```
GET /academic/streams
```

### Create Subject
```
POST /academic/subjects
Body: {
  "name": "Mathematics",
  "code": "MATH",
  "description": "Mathematics subject"
}
```

### Assign Subject to Class
```
POST /academic/classes/:classId/assign-subjects
Body: {
  "subjectIds": ["uuid1", "uuid2"]
}
```

---

## Error Responses

### Format
```json
{
  "statusCode": 400,
  "message": "Error description",
  "error": "BAD_REQUEST"
}
```

### Common Status Codes
- **200**: Success
- **201**: Created
- **400**: Bad Request (validation error)
- **401**: Unauthorized (missing/invalid token)
- **403**: Forbidden (insufficient permissions)
- **404**: Not Found
- **500**: Server Error

---

## Testing with Postman/Thunder Client

### Setup
1. Create new environment variable:
   - `token`: (paste from login response `access_token`)
   - `tenantId`: (your tenant UUID)
   - `apiUrl`: http://localhost:3001

2. Set in all requests:
   - Authorization: `Bearer {{token}}`
   - x-tenant-id: `{{tenantId}}`

### Example Request
```
GET {{apiUrl}}/students
Headers:
  Authorization: Bearer {{token}}
  x-tenant-id: {{tenantId}}
  Content-Type: application/json
```

---

## Frontend Hook Usage

```typescript
// Import hooks
import {
  useStudents,
  useCreateStudent,
  useTeachers,
  useMarks,
  useFinanceData,
  useAttendance,
  useNotifications,
  useLibraryResources
} from '@/hooks/useApi';

// Use in component
function MyComponent() {
  const { data: students, loading, error } = useStudents();
  const { create: createStudent } = useCreateStudent();
  
  return (
    <>
      {loading && <Skeleton />}
      {error && <Error message={error} />}
      {students?.map(s => <StudentCard key={s.id} student={s} />)}
    </>
  );
}
```

---

## Common Flows

### Student Registration Flow
1. `POST /academic/classes` - Create/get class
2. `POST /academic/streams` - Create/get stream
3. `POST /students` - Register student

### Mark Entry Flow
1. `POST /exams` - Create exam
2. `POST /marks` - Enter marks OR `POST /marks/bulk` - Bulk upload
3. `POST /marks/:id/approve` - Approve marks
4. `GET /exams/:id/results` - View results

### Finance Flow
1. `POST /finance/fee-structures` - Setup fees
2. `POST /finance/invoices/generate` - Generate invoices
3. Wait for payments
4. `POST /finance/payments` - Record payments
5. `GET /finance/collections` - View collections

### Attendance Flow
1. Each day: `POST /attendance` - Record attendance
2. `GET /attendance/summary` - View summary

---

**Last Updated**: 2024
**Status**: Complete & Ready for Testing
