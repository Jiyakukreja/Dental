# 🦷 DentWise - Interview Preparation Guide

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Tech Stack Deep Dive](#tech-stack-deep-dive)
3. [System Architecture](#system-architecture)
4. [Database Design](#database-design)
5. [Key Features Implementation](#key-features-implementation)
6. [Authentication & Authorization](#authentication--authorization)
7. [State Management](#state-management)
8. [API Design](#api-design)
9. [Performance Optimizations](#performance-optimizations)
10. [Security Best Practices](#security-best-practices)
11. [Common Interview Questions & Answers](#common-interview-questions--answers)
12. [Technical Challenges & Solutions](#technical-challenges--solutions)
13. [Scalability Considerations](#scalability-considerations)
14. [Testing Strategy](#testing-strategy)
15. [Deployment & DevOps](#deployment--devops)

---

## 🎯 Project Overview

**DentWise** is a modern, full-stack dental appointment booking platform with AI-powered voice assistance. It enables patients to book appointments with dentists, manage their dental health records, and receive instant dental advice through an AI assistant.

### Core Functionalities:
- 🏥 **Patient Dashboard**: View appointments, dental health overview, and activity
- 👨‍⚕️ **Doctor Management**: Admin panel for managing doctors and their profiles
- 📅 **Appointment Booking**: Multi-step booking process with real-time availability
- 🤖 **AI Voice Assistant**: VAPI-powered dental consultation (premium feature)
- 📧 **Email Notifications**: Automated appointment confirmations
- 🔐 **Authentication**: Secure user management with Clerk

### Business Logic:
- Users can book appointments with available doctors
- Admin can manage doctor profiles and view all appointments
- Premium users get access to AI voice assistant for 24/7 dental advice
- Email confirmations sent automatically on booking

---

## 🛠️ Tech Stack Deep Dive

### Frontend Framework
**Next.js 15.5.9** (React 19.1.0)
- **Why Next.js?**
  - Server-Side Rendering (SSR) for better SEO and initial load performance
  - API routes for backend functionality
  - File-based routing system
  - Built-in image optimization
  - React Server Components for reduced client-side JS
  
- **Key Features Used:**
  - `app` directory (App Router)
  - Server Actions for data mutations
  - `force-dynamic` rendering for authentication pages
  - Middleware for route protection

### UI Library
**React 19 + shadcn/ui + Radix UI**
- **Component System:**
  - Headless UI components (Radix) for accessibility
  - Tailwind CSS for styling
  - Custom theme system with CSS variables
  - Responsive design patterns

### Database & ORM
**PostgreSQL + Prisma 6.16.2**
- **Why Prisma?**
  - Type-safe database queries
  - Automatic migration generation
  - Intuitive schema definition
  - Excellent TypeScript integration
  
- **Database Features:**
  - Relational data modeling
  - Cascade deletes for data integrity
  - Unique constraints
  - Indexed queries

### Authentication
**Clerk 6.32.2**
- **Features:**
  - Social login support
  - JWT-based authentication
  - Built-in user management UI
  - Role-based access (plans: ai_basic, ai_pro)
  - Webhook support for user sync

### State Management
**TanStack Query (React Query) 5.89.0**
- **Benefits:**
  - Server state management
  - Automatic caching
  - Background refetching
  - Optimistic updates
  - Query invalidation

### Email Service
**Resend 6.1.0 + React Email**
- Transactional email service
- React-based email templates
- Type-safe email components

### AI Voice Assistant
**VAPI AI 2.3.9**
- Real-time voice conversations
- Custom prompts for dental assistance
- Integration with Next.js

### Form Handling
**React Hook Form 7.63.0 + Zod 4.1.11**
- Type-safe form validation
- Schema-based validation
- Reduced re-renders
- Native HTML validation

### Code Quality
**Biome 2.2.0**
- Linting and formatting
- Replaces ESLint + Prettier
- Faster performance

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Client Browser                       │
│  ┌──────────────────────────────────────────────────┐  │
│  │         Next.js App (React 19)                   │  │
│  │  - Server Components                             │  │
│  │  - Client Components                             │  │
│  │  - Server Actions                                │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ├──────────────────┐
                     │                  │
                     ▼                  ▼
          ┌────────────────┐    ┌──────────────┐
          │  Clerk Auth    │    │  VAPI AI     │
          │  (JWT)         │    │  (Voice)     │
          └────────────────┘    └──────────────┘
                     │
                     ▼
          ┌────────────────────────────┐
          │   Next.js API Routes       │
          │   & Server Actions         │
          │  - /api/send-email         │
          │  - Server Action functions │
          └────────────────────────────┘
                     │
                     ▼
          ┌────────────────────────────┐
          │   Prisma ORM               │
          │   (Type-safe queries)      │
          └────────────────────────────┘
                     │
                     ▼
          ┌────────────────────────────┐
          │   PostgreSQL Database      │
          │   - users                  │
          │   - doctors                │
          │   - appointments           │
          └────────────────────────────┘
                     │
                     ▼
          ┌────────────────────────────┐
          │   Resend Email Service     │
          └────────────────────────────┘
```

### Request Flow:

1. **User Authentication:**
   - User signs in via Clerk
   - JWT token issued
   - Middleware validates on each request
   - User synced to PostgreSQL via server action

2. **Appointment Booking:**
   - Client component collects booking data
   - React Query mutation calls server action
   - Server action validates with Clerk auth
   - Prisma creates appointment in PostgreSQL
   - Email API route sends confirmation via Resend
   - Query invalidation triggers UI refresh

3. **Admin Operations:**
   - Email-based admin check (middleware)
   - Protected routes redirect unauthorized users
   - Server actions for CRUD operations
   - Optimistic updates with React Query

---

## 🗃️ Database Design

### Schema Overview

```prisma
model User {
  id        String @id @default(cuid())
  clerkId   String @unique
  email     String @unique
  firstName String?
  lastName  String?
  phone     String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  appointments Appointment[]
}

model Doctor {
  id          String   @id @default(cuid())
  name        String
  email       String   @unique
  phone       String
  speciality  String
  bio         String?
  imageUrl    String
  gender      Gender
  isActive    Boolean @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  appointments Appointment[]
}

model Appointment {
  id          String            @id @default(cuid())
  date        DateTime
  time        String
  duration    Int               @default(30)
  status      AppointmentStatus @default(CONFIRMED)
  notes       String?
  reason      String?
  createdAt   DateTime          @default(now())
  updatedAt   DateTime          @updatedAt
  userId      String
  doctorId    String
  user        User @relation(fields: [userId], references: [id], onDelete: Cascade)
  doctor      Doctor @relation(fields: [doctorId], references: [id], onDelete: Cascade)
}

enum Gender {
  MALE
  FEMALE
}

enum AppointmentStatus {
  CONFIRMED
  COMPLETED
}
```

### Database Design Decisions:

**1. Separate User and Doctor Models**
- **Why?** Different attributes and business logic
- Users linked to Clerk authentication
- Doctors managed independently by admin
- Clear separation of concerns

**2. Appointment as Junction Table**
- Many-to-many relationship between Users and Doctors
- Additional attributes (date, time, status)
- Cascade deletes maintain referential integrity

**3. String-based Time Storage**
- **Why?** Simplified time slot matching
- Easier for frontend display
- No timezone conversion complexity
- Example: "14:30" format

**4. CUID for IDs**
- Collision-resistant IDs
- Better than auto-increment for distributed systems
- URL-safe

**5. Soft Delete Pattern (isActive)**
- Doctors can be deactivated without deletion
- Maintains historical appointment records
- Better for data integrity and auditing

**6. Enums for Status**
- Type safety at database level
- Prevents invalid status values
- Easy to add new statuses (e.g., CANCELLED)

### Relationships:
- **User → Appointment**: One-to-Many
- **Doctor → Appointment**: One-to-Many
- **Cascade Delete**: Deleting a user/doctor removes their appointments

---

## 🎨 Key Features Implementation

### 1. Multi-Step Appointment Booking

**Component Architecture:**
```
AppointmentBooking (Parent State)
├── ProgressSteps (UI indicator)
├── DoctorSelectionStep
│   └── DoctorCards (useAvailableDoctors)
├── TimeSelectionStep
│   └── Calendar + TimeSlots (useBookedTimeSlots)
└── BookingConfirmationStep
    └── Summary + Submit (useBookAppointment)
```

**State Management:**
```typescript
const [step, setStep] = useState(1);
const [selectedDoctor, setSelectedDoctor] = useState<Doctor | null>(null);
const [selectedDate, setSelectedDate] = useState<Date | undefined>(undefined);
const [selectedTime, setSelectedTime] = useState<string>("");
```

**Key Logic:**
- Real-time availability checking
- Optimistic UI updates
- Form validation at each step
- Error handling with user feedback

### 2. Admin Dashboard

**Features:**
- View all appointments (past and upcoming)
- Add/Edit/Deactivate doctors
- Statistics overview
- Email-based authorization

**Authorization Pattern:**
```typescript
const user = await currentUser();
const adminEmail = process.env.ADMIN_EMAIL;
const userEmail = user.emailAddresses[0]?.emailAddress;

if (userEmail !== adminEmail) redirect("/dashboard");
```

**Why Environment-Based?**
- Simple for single admin
- No complex role management needed
- Easy to change without code updates
- Secure (not hardcoded)

### 3. Real-Time Availability System

**Implementation:**
```typescript
export function useBookedTimeSlots(doctorId: string, date: string) {
  return useQuery({
    queryKey: ["getBookedTimeSlots", doctorId, date],
    queryFn: () => getBookedTimeSlots(doctorId, date),
    enabled: !!doctorId && !!date,
  });
}
```

**Server Action:**
```typescript
export async function getBookedTimeSlots(doctorId: string, date: string) {
  const appointments = await prisma.appointment.findMany({
    where: {
      doctorId,
      date: new Date(date),
      status: { in: ["CONFIRMED", "COMPLETED"] },
    },
    select: { time: true },
  });
  return appointments.map((appointment) => appointment.time);
}
```

**How It Works:**
1. User selects doctor and date
2. Query fetches booked slots for that combination
3. UI disables booked time slots
4. Real-time updates via query refetch

### 4. AI Voice Assistant (VAPI Integration)

**Premium Feature Gating:**
```typescript
const { has } = await auth();
const hasProPlan = has({ plan: "ai_basic" }) || has({ plan: "ai_pro" });

if (!hasProPlan) return <ProPlanRequired />;
```

**VAPI Setup:**
```typescript
import Vapi from "@vapi-ai/web";
export const vapi = new Vapi(process.env.NEXT_PUBLIC_VAPI_API_KEY as string);
```

**Custom Prompt Engineering:**
- Dental-specific knowledge base
- Service pricing information
- Emergency handling guidelines
- Appointment booking redirects
- Empathetic conversation flow

**Key Prompt Sections:**
- Identity & Purpose
- Voice & Persona
- Conversation Flow
- Service Capabilities
- Scenario Handling (Emergency, Routine, Cost)

### 5. Email Notification System

**Architecture:**
```typescript
// React Email Component
<AppointmentConfirmationEmail
  doctorName={doctor.name}
  appointmentDate={formattedDate}
  appointmentTime={time}
  appointmentType={reason}
  duration="30 minutes"
  price={servicePrice}
/>

// API Route (app/api/send-appointment-email/route.ts)
const { data, error } = await resend.emails.send({
  from: "DentWise <appointments@dentwise.com>",
  to: [userEmail],
  subject: "Your DentWise Appointment is Confirmed!",
  react: AppointmentConfirmationEmail({...}),
});
```

**Email Template Features:**
- Responsive HTML design
- Branded styling
- Appointment details
- Calendar integration option
- Support contact information

---

## 🔐 Authentication & Authorization

### Clerk Integration

**Middleware Protection:**
```typescript
import { clerkMiddleware } from "@clerk/nextjs/server";

export default clerkMiddleware();

export const config = {
  matcher: [
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

**User Sync Pattern:**
```typescript
export async function syncUser() {
  const user = await currentUser();
  if (!user) return;
  
  const existingUser = await prisma.user.findUnique({ 
    where: { clerkId: user.id } 
  });
  
  if (existingUser) return existingUser;
  
  // Create new user in database
  const dbUser = await prisma.user.create({
    data: {
      clerkId: user.id,
      firstName: user.firstName,
      lastName: user.lastName,
      email: user.emailAddresses[0].emailAddress,
      phone: user.phoneNumbers[0]?.phoneNumber,
    },
  });
  
  return dbUser;
}
```

**Why This Approach?**
- Keeps Clerk as single source of truth
- Database stores only necessary data
- Webhook-based sync possible for production
- Simple and reliable

### Authorization Levels

1. **Public Routes**: Landing page, pricing
2. **Authenticated Routes**: Dashboard, appointments
3. **Admin Routes**: Doctor management, all appointments
4. **Premium Routes**: AI voice assistant

**Route Protection Pattern:**
```typescript
// Server Component
export const dynamic = 'force-dynamic';

async function ProtectedPage() {
  const user = await currentUser();
  if (!user) redirect("/");
  
  // ... page content
}
```

---

## 📊 State Management

### TanStack Query (React Query) Implementation

**Query Pattern (Read Operations):**
```typescript
export function useGetDoctors() {
  return useQuery({
    queryKey: ["getDoctors"],
    queryFn: getDoctors,
  });
}
```

**Mutation Pattern (Write Operations):**
```typescript
export function useBookAppointment() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: bookAppointment,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["getUserAppointments"] });
    },
    onError: (error) => console.error("Failed to book appointment:", error),
  });
}
```

**Optimistic Updates:**
```typescript
const { mutate: updateStatus } = useUpdateAppointmentStatus();

// Optimistically update UI before server responds
updateStatus({ id: appointmentId, status: "COMPLETED" });
```

**Key Benefits:**
- Automatic caching (reduces API calls)
- Background refetching (data always fresh)
- Request deduplication
- Stale-while-revalidate pattern
- Loading and error states handled automatically

**Query Key Strategy:**
- `["getDoctors"]` - All doctors
- `["getAvailableDoctors"]` - Active doctors only
- `["getBookedTimeSlots", doctorId, date]` - Scoped to specific doctor/date
- `["getUserAppointments"]` - User-specific data

---

## 🔌 API Design

### Server Actions (Preferred Approach)

**Why Server Actions?**
- Type-safe end-to-end
- No API route boilerplate
- Automatic serialization
- Progressive enhancement

**Example: Book Appointment**
```typescript
"use server";

interface BookAppointmentInput {
  doctorId: string;
  date: string;
  time: string;
  reason?: string;
}

export async function bookAppointment(input: BookAppointmentInput) {
  try {
    // 1. Authenticate
    const { userId } = await auth();
    if (!userId) throw new Error("You must be logged in");
    
    // 2. Validate
    if (!input.doctorId || !input.date || !input.time) {
      throw new Error("Doctor, date, and time are required");
    }
    
    // 3. Get user from database
    const user = await prisma.user.findUnique({ where: { clerkId: userId } });
    if (!user) throw new Error("User not found");
    
    // 4. Create appointment
    const appointment = await prisma.appointment.create({
      data: {
        userId: user.id,
        doctorId: input.doctorId,
        date: new Date(input.date),
        time: input.time,
        reason: input.reason || "General consultation",
        status: "CONFIRMED",
      },
      include: {
        user: { select: { firstName: true, lastName: true, email: true } },
        doctor: { select: { name: true, imageUrl: true } },
      },
    });
    
    // 5. Send email notification
    await fetch("/api/send-appointment-email", {
      method: "POST",
      body: JSON.stringify({ appointment }),
    });
    
    return appointment;
  } catch (error) {
    console.error("Error booking appointment:", error);
    throw new Error("Failed to book appointment");
  }
}
```

### API Routes (For External Services)

**Email API Route:**
```typescript
// app/api/send-appointment-email/route.ts
export async function POST(request: Request) {
  try {
    const { appointment } = await request.json();
    
    const { data, error } = await resend.emails.send({
      from: "DentWise <appointments@dentwise.com>",
      to: [appointment.user.email],
      subject: "Your DentWise Appointment is Confirmed!",
      react: AppointmentConfirmationEmail({
        doctorName: appointment.doctor.name,
        appointmentDate: appointment.date,
        appointmentTime: appointment.time,
        appointmentType: appointment.reason,
        duration: "30 minutes",
        price: "$120",
      }),
    });
    
    if (error) throw error;
    
    return Response.json({ success: true, data });
  } catch (error) {
    return Response.json({ error: "Failed to send email" }, { status: 500 });
  }
}
```

**When to Use Each:**
- **Server Actions**: Database operations, form submissions
- **API Routes**: Webhooks, external service integration, third-party APIs

---

## ⚡ Performance Optimizations

### 1. Server Components by Default
```typescript
// No "use client" = Server Component
async function DashboardPage() {
  // Runs on server, no client JS sent
  const appointments = await getAppointments();
  return <div>{/* JSX */}</div>;
}
```

**Benefits:**
- Reduced client-side JavaScript
- Direct database access
- No prop drilling
- Automatic code splitting

### 2. Dynamic Rendering
```typescript
export const dynamic = 'force-dynamic';
```
- Prevents static generation
- Always fresh data
- Important for user-specific content

### 3. Image Optimization
```typescript
// next.config.ts
images: {
  remotePatterns: [
    { protocol: "https", hostname: "images.unsplash.com" },
    { protocol: "https", hostname: "avatar.iran.liara.run" },
  ],
  unoptimized: true,
}
```

### 4. Query Caching Strategy
```typescript
useQuery({
  queryKey: ["getDoctors"],
  queryFn: getDoctors,
  staleTime: 5 * 60 * 1000, // 5 minutes
  cacheTime: 10 * 60 * 1000, // 10 minutes
});
```

### 5. Parallel Data Fetching
```typescript
const [totalCount, completedCount] = await Promise.all([
  prisma.appointment.count({ where: { userId: user.id } }),
  prisma.appointment.count({ where: { userId: user.id, status: "COMPLETED" } }),
]);
```

### 6. Conditional Queries
```typescript
useQuery({
  queryKey: ["getBookedTimeSlots"],
  queryFn: () => getBookedTimeSlots(doctorId, date),
  enabled: !!doctorId && !!date, // Only run when both are present
});
```

---

## 🛡️ Security Best Practices

### 1. Authentication on Every Request
```typescript
const { userId } = await auth();
if (!userId) throw new Error("Unauthorized");
```

### 2. Input Validation
```typescript
if (!input.doctorId || !input.date || !input.time) {
  throw new Error("Doctor, date, and time are required");
}
```

### 3. Database-Level Constraints
```prisma
email String @unique
clerkId String @unique
```

### 4. Cascade Deletes
```prisma
user User @relation(fields: [userId], references: [id], onDelete: Cascade)
```
- Prevents orphaned records
- Maintains data integrity

### 5. Environment Variables
```typescript
process.env.ADMIN_EMAIL
process.env.NEXT_PUBLIC_VAPI_API_KEY
process.env.DATABASE_URL
```

### 6. Admin Authorization
```typescript
const adminEmail = process.env.ADMIN_EMAIL;
const userEmail = user.emailAddresses[0]?.emailAddress;
if (userEmail !== adminEmail) redirect("/dashboard");
```

### 7. Error Handling
```typescript
try {
  // Operation
} catch (error) {
  console.error("Error:", error);
  throw new Error("User-friendly error message");
}
```

---

## 💬 Common Interview Questions & Answers

### Architecture & Design

**Q: Why did you choose Next.js over Create React App?**
> **A:** Next.js provides several advantages for this project:
> 1. **Server-Side Rendering**: Better SEO for the landing page and initial load performance
> 2. **Server Actions**: Type-safe backend logic without API route boilerplate
> 3. **File-based Routing**: Intuitive and scalable route structure
> 4. **Server Components**: Reduced client-side JS, direct database access
> 5. **Built-in API Routes**: Easy to create endpoints for email service
> 6. **Middleware**: Perfect for authentication checks on every request

**Q: How does your authentication flow work?**
> **A:** 
> 1. User signs in via Clerk (handles OAuth, passwords, sessions)
> 2. Clerk issues JWT token, stored in cookies
> 3. Middleware validates token on every request
> 4. `syncUser` function creates/updates user in PostgreSQL
> 5. Server Actions use `auth()` to get userId from Clerk
> 6. Map clerkId to internal user ID for database operations
> 
> This separates authentication (Clerk) from authorization (our database), making it scalable.

**Q: Why use Prisma instead of raw SQL?**
> **A:**
> 1. **Type Safety**: Auto-generated TypeScript types from schema
> 2. **Migration Management**: Easy to track and roll back changes
> 3. **Developer Experience**: Intuitive API, auto-completion
> 4. **Prevention of SQL Injection**: Parameterized queries
> 5. **Relationship Management**: Simplified includes and joins
> 
> Example: `prisma.appointment.findMany({ include: { user: true, doctor: true } })`
> vs complex SQL joins with manual type definitions.

**Q: How do you prevent double-booking appointments?**
> **A:** 
> 1. Fetch booked time slots for selected doctor and date
> 2. Disable those slots in UI (client-side prevention)
> 3. Could add unique constraint on (doctorId, date, time) for database-level prevention
> 4. Transaction-based booking for production (row-level locking)
> 
> Current implementation focuses on UI prevention, but I'd add database constraints for production.

### State Management

**Q: Why TanStack Query instead of Redux?**
> **A:** 
> TanStack Query is specifically designed for server state, which is 90% of our state:
> 1. **Automatic Caching**: Reduces API calls
> 2. **Background Refetching**: Data stays fresh
> 3. **Less Boilerplate**: No actions, reducers, sagas
> 4. **Built-in Loading/Error States**: Simplified error handling
> 5. **Optimistic Updates**: Better UX
> 
> Redux is better for complex client state with many interactions. Our app has minimal client state (form inputs, UI toggles), which React's useState handles fine.

**Q: Explain your query invalidation strategy.**
> **A:**
> ```typescript
> const { mutate } = useMutation({
>   mutationFn: bookAppointment,
>   onSuccess: () => {
>     queryClient.invalidateQueries({ queryKey: ["getUserAppointments"] });
>   },
> });
> ```
> When appointment is booked, we invalidate the getUserAppointments query, triggering automatic refetch. This ensures the dashboard shows the new appointment immediately.
> 
> Alternative approaches: optimistic updates, manual cache updates.

### Database & Backend

**Q: Why separate User and Doctor models?**
> **A:**
> 1. **Different Lifecycles**: Users sign up themselves, doctors are admin-managed
> 2. **Different Attributes**: Doctors have speciality, bio, isActive; Users have Clerk integration
> 3. **Authorization**: Only admins can create doctors
> 4. **Scalability**: Easier to add doctor-specific features (schedules, ratings)

**Q: How would you handle appointment cancellations?**
> **A:**
> Current schema has CONFIRMED and COMPLETED. To add cancellation:
> 1. Add CANCELLED to AppointmentStatus enum
> 2. Create cancelAppointment server action
> 3. Add cancelledAt timestamp and cancellationReason fields
> 4. Update UI to show cancel button for future appointments
> 5. Send cancellation email notification
> 6. Free up the time slot for other patients

**Q: How do you handle timezone differences?**
> **A:**
> Currently, times are stored as strings ("14:30"). For production:
> 1. Store doctor's timezone in Doctor model
> 2. Use UTC for date field in database
> 3. Convert to doctor's timezone for display
> 4. Use date-fns-tz or Luxon for conversions
> 5. Show timezone to user during booking
> 
> Current implementation assumes single timezone (simplified MVP).

### Performance

**Q: How would you optimize for 10,000 concurrent users?**
> **A:**
> 1. **Database**: 
>    - Add indexes on frequently queried columns (doctorId, date, userId)
>    - Connection pooling (Prisma handles this)
>    - Read replicas for heavy read operations
> 
> 2. **Caching**:
>    - Redis for session storage
>    - Cache doctor list (rarely changes)
>    - CDN for static assets
> 
> 3. **Query Optimization**:
>    - Pagination for appointment lists
>    - Lazy loading for components
>    - Debounce search inputs
> 
> 4. **Infrastructure**:
>    - Horizontal scaling (multiple server instances)
>    - Load balancer
>    - Edge functions for API routes

**Q: Current performance bottlenecks?**
> **A:**
> 1. **Email Sending**: Blocks mutation completion. Solution: Background job queue
> 2. **No Pagination**: All appointments loaded at once. Solution: Infinite scroll or pagination
> 3. **Image Loading**: External avatars. Solution: Next.js Image optimization
> 4. **Client-Side Filtering**: For doctors/appointments. Solution: Server-side filtering with query params

### AI Integration

**Q: How does the VAPI voice assistant work?**
> **A:**
> 1. Premium users get access via Clerk plan check
> 2. VAPI widget initialized with API key
> 3. Custom prompt defines Riley's personality and capabilities
> 4. Real-time voice conversation via WebRTC
> 5. Prompt includes service pricing, dental advice, emergency handling
> 6. Cannot book appointments (redirects to booking page)
> 
> Key design decision: Voice AI for consultation, not booking (requires payment security).

**Q: How would you improve the AI assistant?**
> **A:**
> 1. **Context Awareness**: Access user's appointment history
> 2. **Multilingual Support**: Detect and respond in user's language
> 3. **Sentiment Analysis**: Detect urgency, provide appropriate recommendations
> 4. **Follow-up**: Send email summaries of conversation
> 5. **Integration**: Direct booking after consultation with user confirmation

### Security

**Q: How do you prevent unauthorized access to admin panel?**
> **A:**
> ```typescript
> const adminEmail = process.env.ADMIN_EMAIL;
> const userEmail = user.emailAddresses[0]?.emailAddress;
> if (userEmail !== adminEmail) redirect("/dashboard");
> ```
> Email-based check. For production with multiple admins:
> 1. Create Admin model in database
> 2. Many-to-many relationship: User ↔ Admin
> 3. Middleware check for admin routes
> 4. Or use Clerk's organization/role features

**Q: How do you prevent SQL injection?**
> **A:**
> Prisma uses parameterized queries automatically. All user input is sanitized:
> ```typescript
> // This is safe:
> prisma.appointment.findMany({ where: { doctorId: userInput } })
> 
> // Prisma converts to: SELECT * FROM appointments WHERE doctorId = $1
> ```
> Never constructing raw SQL with template literals.

### Testing

**Q: What's your testing strategy?**
> **A:**
> Current project doesn't have tests (MVP), but production strategy:
> 1. **Unit Tests**: Server Actions, utility functions (Jest)
> 2. **Integration Tests**: API routes, database operations (Jest + Prisma mock)
> 3. **E2E Tests**: Critical flows (Playwright/Cypress)
>    - User registration → booking → email confirmation
>    - Admin creates doctor → user books with that doctor
> 4. **Component Tests**: React Testing Library for UI components
> 
> Priority: Test business-critical paths first (booking, payment if added).

---

## 🔧 Technical Challenges & Solutions

### Challenge 1: Real-Time Availability
**Problem**: Show only available time slots for selected doctor/date
**Solution**:
- Conditional query with useQuery's `enabled` option
- Query runs only when doctor AND date selected
- UI disables booked slots
- Background refetch keeps data fresh

### Challenge 2: User Sync with Clerk
**Problem**: Keep PostgreSQL user data in sync with Clerk
**Solution**:
- `syncUser` server action on landing page
- Checks if user exists by clerkId
- Creates if not exists, returns if exists
- Could be replaced with Clerk webhooks for real-time sync

### Challenge 3: Email Notifications
**Problem**: Send confirmation emails after booking
**Solution**:
- React Email for type-safe templates
- Resend for delivery
- API route to separate concerns
- Could be improved with queue (Bull/BullMQ) for reliability

### Challenge 4: Admin Authorization
**Problem**: Simple admin access without complex role system
**Solution**:
- Environment variable for admin email
- Server-side check on admin pages
- Scales to multiple admins with database approach

### Challenge 5: Type Safety Across Stack
**Problem**: Maintain types from DB → Server → Client
**Solution**:
- Prisma generates types from schema
- Server Actions inherit Prisma types
- Client hooks typed from Server Actions
- Zod for runtime validation

---

## 📈 Scalability Considerations

### Current Limitations & Solutions:

1. **Single Admin**
   - **Current**: Environment variable
   - **Scale**: Admin model with role-based access control

2. **No Pagination**
   - **Current**: Load all appointments
   - **Scale**: Cursor-based pagination, infinite scroll

3. **Synchronous Email**
   - **Current**: Blocks mutation
   - **Scale**: Message queue (Redis + Bull)

4. **No Caching Layer**
   - **Current**: Query-level caching
   - **Scale**: Redis for session, frequently accessed data

5. **Single Database**
   - **Current**: PostgreSQL
   - **Scale**: Read replicas, sharding by geographic region

6. **No Rate Limiting**
   - **Current**: None
   - **Scale**: Middleware-based rate limiting, Upstash Redis

7. **No Monitoring**
   - **Current**: Console logs
   - **Scale**: Sentry for errors, Vercel Analytics

### Horizontal Scaling Plan:
```
Load Balancer
├── Next.js Instance 1
├── Next.js Instance 2
└── Next.js Instance 3
        ↓
   PostgreSQL Primary
        ↓
   Read Replicas (2+)
        ↓
   Redis Cache
        ↓
   Background Job Queue
```

---

## 🧪 Testing Strategy

### Unit Tests:
```typescript
describe("bookAppointment", () => {
  it("should create appointment with valid data", async () => {
    const input = {
      doctorId: "doc-123",
      date: "2026-02-10",
      time: "14:30",
      reason: "Checkup"
    };
    
    const result = await bookAppointment(input);
    
    expect(result.status).toBe("CONFIRMED");
    expect(result.doctorId).toBe(input.doctorId);
  });
  
  it("should throw error for unauthenticated user", async () => {
    // Mock auth to return null
    await expect(bookAppointment(input)).rejects.toThrow("You must be logged in");
  });
});
```

### Integration Tests:
```typescript
describe("Appointment Booking Flow", () => {
  it("should book appointment and send email", async () => {
    // 1. Create test user
    // 2. Create test doctor
    // 3. Book appointment
    // 4. Verify database record
    // 5. Verify email sent
  });
});
```

### E2E Tests (Playwright):
```typescript
test("User can book appointment", async ({ page }) => {
  await page.goto("/");
  await page.click("text=Sign In");
  // ... authentication flow
  await page.click("text=Book Appointment");
  await page.click("[data-testid=doctor-card-1]");
  await page.click("[data-testid=time-slot-14:30]");
  await page.click("text=Confirm Booking");
  await expect(page.locator("text=Appointment Confirmed")).toBeVisible();
});
```

---

## 🚀 Deployment & DevOps

### Current Setup (Assumed):
- **Hosting**: Vercel (recommended for Next.js)
- **Database**: Vercel Postgres / Supabase / Neon
- **Email**: Resend
- **Authentication**: Clerk
- **AI**: VAPI

### CI/CD Pipeline:
```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      - run: npm run lint
      - run: npm run build
      - run: npx prisma generate
      - run: npx prisma migrate deploy
      - uses: vercel/deploy-action@v1
```

### Environment Variables:
```env
# Database
DATABASE_URL="postgresql://..."

# Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="..."
CLERK_SECRET_KEY="..."

# Email
RESEND_API_KEY="..."

# AI
NEXT_PUBLIC_VAPI_API_KEY="..."

# Admin
ADMIN_EMAIL="admin@dentwise.com"
```

### Database Migrations:
```bash
# Development
npx prisma migrate dev --name add_cancellation_status

# Production
npx prisma migrate deploy
```

---

## 🎓 Key Takeaways for Interview

### What You Built:
✅ Full-stack appointment booking system
✅ Real-time availability checking
✅ AI-powered voice assistant
✅ Admin dashboard with CRUD operations
✅ Email notification system
✅ Secure authentication & authorization

### Technologies Mastered:
✅ Next.js 15 (App Router, Server Components, Server Actions)
✅ React 19 (latest features)
✅ TypeScript (end-to-end type safety)
✅ Prisma (ORM, migrations)
✅ PostgreSQL (relational database design)
✅ TanStack Query (server state management)
✅ Clerk (authentication)
✅ VAPI (AI integration)
✅ React Email + Resend (transactional emails)

### Soft Skills Demonstrated:
✅ System design thinking
✅ Problem-solving (real-time slots, user sync)
✅ Security awareness (auth, validation)
✅ Performance optimization (caching, parallel queries)
✅ Scalability planning (pagination, queueing)
✅ User experience focus (multi-step forms, loading states)

### What You Can Discuss:
✅ Why you chose each technology
✅ Trade-offs you made
✅ How you'd improve for production
✅ Challenges you faced and solved
✅ How you'd scale the system
✅ Alternative approaches you considered

---

## 📚 Further Reading & Resources

### Next.js:
- [Next.js Documentation](https://nextjs.org/docs)
- [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [App Router Migration](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration)

### Prisma:
- [Prisma Schema Reference](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference)
- [Database Relationships](https://www.prisma.io/docs/concepts/components/prisma-schema/relations)

### TanStack Query:
- [React Query Documentation](https://tanstack.com/query/latest/docs/react/overview)
- [Optimistic Updates](https://tanstack.com/query/latest/docs/react/guides/optimistic-updates)

### Clerk:
- [Clerk Next.js Integration](https://clerk.com/docs/quickstarts/nextjs)

---

## 🔑 Quick Reference Commands

```bash
# Development
npm run dev

# Build
npm run build

# Database
npx prisma generate
npx prisma migrate dev
npx prisma studio

# Linting
npm run lint
npm run format
```

---

**Good Luck with Your Interview! 🚀**

Remember: Focus on explaining your *thinking process* and *decision-making*, not just the code. Interviewers want to see how you approach problems and make trade-offs.
