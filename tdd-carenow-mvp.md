# **Technical Design Document: CareNow MVP**

### **1\. Introduction & Purpose**

This document details the technical design and architecture for the CareNow MVP. Its purpose is to provide a clear technical blueprint for developers, ensuring that the implementation aligns with the project's requirements, scalability goals, and security considerations.

### **2\. PRD Reference**

This TDD is based on the finalized Product Requirements Document: prd-carenow-mvp.md.

### **3\. Technical Goals**

* Implement a secure, multi-tenant architecture on Supabase PostgreSQL with row-level security policies.  
* Develop a decoupled frontend (React) and backend (Node.js/Express) to ensure a clean separation of concerns.  
* Integrate third-party services (Paystack, Clickatell, Sentry) through a modular, service-oriented backend structure.  
* Ensure the entire application is containerized with Docker for development parity and simplified deployment.

### **4\. Technical Overview & Architecture**

The system will be composed of a React frontend, a Node.js backend API, a Supabase instance for database and authentication, and a Redis instance for job queuing.

* **Frontend:** A React (Vite) single-page application using TypeScript, Tailwind CSS, and Shadcn/UI. It will communicate directly with Supabase for authentication and with our backend API for business logic.  
* **Backend API:** A Node.js/Express server written in TypeScript. It will handle complex business logic, serve as a proxy for secure operations, and process webhooks from third-party services.  
* **Database & Auth:** Supabase will provide the PostgreSQL database and user authentication. Prisma will be the ORM used by the backend to communicate with the database. JWTs issued by Supabase will be used to secure the backend API.  
* **Job Queue:** BullMQ with Redis will manage background jobs, primarily for scheduling and sending 24-hour SMS/email notifications.

### **5\. Data Model Design**

The following Prisma schema outlines the core data models. tenantId will be used on relevant tables to enforce multi-tenancy.

// Tenant represents a single clinic/practice  
model Tenant {  
  id                 String        @id @default(cuid())  
  name               String  
  status             TenantStatus  @default(ACTIVE)  
  subscriptionPlan   Plan          @default(SOLO)  
  paystackCustomerId String?       @unique  
  createdAt          DateTime      @default(now())  
  updatedAt          DateTime      @updatedAt  
  profiles           Profile\[\]  
  patients           Patient\[\]  
  appointments       Appointment\[\]  
}

// Profile links a User to a Tenant and their role  
model Profile {  
  id        String   @id @default(uuid()) // Corresponds to Supabase auth.users.id  
  tenantId  String  
  role      UserRole  
  firstName String  
  lastName  String  
  tenant    Tenant   @relation(fields: \[tenantId\], references: \[id\])  
  doctor    Doctor?  
}

// Patient profile, scoped to a tenant  
model Patient {  
  id                String        @id @default(cuid())  
  tenantId          String  
  // ... fields from PRD (name, surname, medicalAid, etc.)  
  tenant            Tenant        @relation(fields: \[tenantId\], references: \[id\])  
  appointments      Appointment\[\]  
}

// Doctor specific information  
model Doctor {  
  id            String         @id @default(cuid())  
  profileId     String         @unique  
  specialty     String?  
  profile       Profile        @relation(fields: \[profileId\], references: \[id\])  
  availabilities Availability\[\]  
  appointments  Appointment\[\]  
}

model Availability {  
  id        String    @id @default(cuid())  
  doctorId  String  
  dayOfWeek Int       // 0 \= Sunday, 1 \= Monday, etc.  
  startTime DateTime  // Time of day  
  endTime   DateTime  // Time of day  
  doctor    Doctor    @relation(fields: \[doctorId\], references: \[id\])  
}

model Appointment {  
  id          String            @id @default(cuid())  
  tenantId    String  
  patientId   String  
  doctorId    String  
  startTime   DateTime  
  endTime     DateTime  
  status      AppointmentStatus @default(PENDING)  
  tenant      Tenant            @relation(fields: \[tenantId\], references: \[id\])  
  patient     Patient           @relation(fields: \[patientId\], references: \[id\])  
  doctor      Doctor            @relation(fields: \[doctorId\], references: \[id\])  
}

enum Plan { SOLO, GROWTH, PRO }  
enum UserRole { ADMIN, DOCTOR }  
enum TenantStatus { ACTIVE, PAST\_DUE, CANCELLED }  
enum AppointmentStatus { PENDING, CONFIRMED, CANCELLED, COMPLETED }

### **6\. API Design (if applicable)**

The backend will expose a RESTful API. All endpoints (except auth/public) will require a valid Supabase JWT.

* POST /api/auth/register: Creates a new tenant, user, and Paystack customer. Initiates subscription.  
* POST /api/tenants/invite: (Admin) Sends an invitation link to a new doctor.  
* GET /api/doctors: (Patient) Lists doctors for a specific tenant.  
* GET /api/doctors/:id/availability: (Patient) Retrieves calculated, bookable time slots for a doctor.  
* GET, POST /api/appointments: (Patient) Create and view appointments.  
* GET, PUT /api/admin/appointments: (Admin/Doctor) Manage all tenant appointments (approve, cancel).  
* POST /api/webhooks/paystack: Handles subscription events from Paystack (e.g., charge.success, subscription.disable).

### **7\. Module/Component Breakdown**

* **Backend Modules:** auth/ (middleware), routes/, controllers/, services/ (e.g., subscriptionService, notificationService), jobs/ (workers for the notification queue).  
* **Frontend Components:** pages/ (Landing, Login, PatientDashboard, AdminDashboard), components/ (AppointmentCalendar, BookingForm, DataTable), api/ (client for interacting with the backend API).

### **8\. Error Handling & Logging Strategy**

* **Error Reporting:** Sentry will be used for real-time error monitoring on both the frontend and backend.  
* **API Errors:** The backend API will return standardized JSON error objects and use appropriate HTTP status codes.  
* **Logging:** In development, logs will be printed to the console. In production, logs will be structured (JSON) for easier parsing by a logging service.

### **9\. Security Considerations**

* **Authentication:** Handled by Supabase Auth. JWTs will be stored in secure, httpOnly cookies.  
* **Authorization:** API endpoints will be protected by middleware that validates the JWT and checks user roles and tenant access.  
* **Data Security:** All sensitive patient data will be encrypted at rest by default in Supabase. Input validation will be enforced on all API endpoints to prevent injection attacks. All secrets will be managed via environment variables.

### **10\. Performance & Scalability Considerations**

* **Database:** Proper indexing will be applied to foreign keys and frequently queried columns (e.g., tenantId, doctorId, appointment dates) to ensure fast lookups.  
* **Backend:** The backend API will be stateless, allowing for horizontal scaling if traffic increases.  
* **Async Tasks:** Using Redis/BullMQ for notifications prevents blocking the main API thread and ensures reminders are sent reliably.

### **11\. Testing Strategy**

* **Unit Tests:** Jest will be used for testing critical business logic within the backend services (e.g., availability calculation, subscription status logic).  
* **Integration Tests:** API endpoints will be tested using a framework like Supertest to ensure controllers, services, and the database work correctly together.  
* **End-to-End (E2E) Tests:** Key user flows (e.g., patient booking, admin approval) will be manually tested for the MVP. Automated E2E testing (e.g., Cypress) is a post-MVP goal.

### **12\. Deployment Considerations (Optional)**

* **Infrastructure:** A single VPS (e.g., Hetzner, DigitalOcean) running Docker.  
* **Containerization:** A docker-compose.yml file will define and link the services: nginx, backend-api, redis. The frontend will be built into a static asset folder and served by Nginx.  
* **SSL:** Nginx will be configured to use Let's Encrypt for free SSL certificates, enabling HTTPS.  
* **CI/CD:** A post-MVP goal is to set up GitHub Actions to automate testing and deployment.

### **13\. Technical Risks & Mitigation Plan**

* **Risk:** Complexity of Paystack webhook integration. **Mitigation:** Use a tool like ngrok for local development and testing of webhooks. Implement a dead-letter queue for failed webhook events to allow for manual reprocessing.  
* **Risk:** Vendor lock-in with Supabase. **Mitigation:** Supabase uses standard PostgreSQL, making migration possible. Using Prisma as an ORM further abstracts the database, simplifying a potential future migration.

### **14\. Out of Scope (Technical Non-Goals)**

* Microservices architecture. The MVP will be a monolith backend API for simplicity.  
* Kubernetes or other complex container orchestration. Docker Compose is sufficient.  
* Real-time features using WebSockets (e.g., live schedule updates).

### **15\. Open Technical Questions**

* None at this stage. Specific implementation details will be resolved during the development sprints.