## **Relevant Files**

* prisma/schema.prisma \- Defines the database schema, models, and relationships using Prisma ORM.  
* prisma/seed.ts \- A script to populate the database with initial data (e.g., a demo clinic, doctor, and patient).  
* backend/src/index.ts \- The main entry point for the Node.js/Express backend server.  
* backend/src/middleware/auth.ts \- Middleware to validate Supabase JWTs and protect API routes.  
* backend/src/services/paystackService.ts \- Service module to handle all interactions with the Paystack API for subscriptions.  
* backend/src/services/notificationService.ts \- Service module for handling email (SendGrid) and SMS (Clickatell) logic.  
* backend/src/jobs/notificationProcessor.ts \- The BullMQ job worker that processes queued notification tasks (e.g., 24-hour reminders).  
* frontend/src/api/client.ts \- A centralized client (e.g., Axios or fetch wrapper) for making requests to the backend API.  
* frontend/src/hooks/useAuth.ts \- A custom React hook to manage user authentication state via Supabase Auth.  
* frontend/src/pages/RegisterClinic.tsx \- The main page for the new tenant/clinic registration and subscription flow.  
* frontend/src/components/AppointmentCalendar.tsx \- A reusable React component for displaying doctor availability and booking appointments.  
* docker-compose.yml \- Defines and configures all the services (backend, nginx, postgres, redis) needed to run the application.  
* nginx/default.conf \- Nginx configuration file for acting as a reverse proxy and serving the frontend static assets.  
* README.md \- The main project documentation, including local setup and deployment instructions.

### **Notes**

* Unit tests should typically be placed alongside the code files they are testing (e.g., myService.ts and myService.test.ts in the same directory).  
* Consider TDD specifications for test coverage or specific test types.  
* Use npm test or npx jest \[optional/path/to/test/file\] to run tests. Running without a path executes all tests found by the Jest configuration.

## **Tasks**

* 1.0 Project Scaffolding & Database Setup  
  * x  
    1.1 Initialize a monorepo structure (e.g., using npm workspaces) with backend and frontend packages.  
  * x  
    1.2 Set up the backend project with Node.js, Express, and TypeScript.  
  * x  
    1.3 Set up the frontend project with React, Vite, and TypeScript.  
  * 1.4 Configure Prisma ORM in the backend, connecting it to the Supabase PostgreSQL database.  
  * 1.5 Define all database models (Tenant, Profile, Patient, Doctor, Availability, Appointment) in prisma/schema.prisma as per the TDD.  
  * 1.6 Generate and run the initial database migration to create the tables.  
  * 1.7 Create a seed script (prisma/seed.ts) to populate the database with a demo clinic, admin, doctor, and patient.  
* 2.0 Backend Core: Tenancy & Subscription API  
  * 2.1 Set up the basic Express server structure with routes, controllers, and services directories.  
  * 2.2 Implement JWT validation middleware to protect routes using JWTs from Supabase Auth.  
  * 2.3 Create the public registration endpoint (POST /api/auth/register) for new tenants.  
  * 2.4 Integrate the Paystack Node.js SDK and configure it with environment variables.  
  * 2.5 Implement the business logic in the registration endpoint to create a Tenant, User, and a Paystack Customer and then subscribe them to a plan.  
  * 2.6 Create the Paystack webhook endpoint (POST /api/webhooks/paystack) to handle subscription status updates.  
  * 2.7 Implement feature-gating middleware that.  
* 3.0 Backend Features: Scheduling & Appointment API  
  * 3.1 Create API endpoints for inviting doctors and managing their profiles.  
  * 3.2 Create API endpoints for doctors to define and update their weekly Availability.  
  * 3.3 Implement the core business logic to calculate available appointment slots based on a doctor's schedule, existing appointments, and buffer times.  
  * 3.4 Create the full set of CRUD API endpoints for Appointments (for patients, doctors, and admins).  
  * 3.5 Implement the approval workflow logic, allowing doctors/admins to change an appointment's status from PENDING to CONFIRMED.  
  * 3.6 Set up Redis and BullMQ and create a queue for processing notifications.  
  * 3.7 Create a job scheduler that adds a notification job to the queue when an appointment is confirmed for 24 hours before the appointment time.  
  * 3.8 Implement the job processor that sends emails via SendGrid and SMS messages via Clickatell.  
* 4.0 Frontend Development: UI & User Flows  
  * 4.1 Set up Shadcn/UI and configure the theme for the React application.  
  * 4.2 Create a reusable API client service to interact with the backend.  
  * 4.3 Implement the main landing page.  
  * 4.4 Implement the authentication pages (Login, Signup) using the Supabase Auth client.  
  * 4.5 Build the multi-step "Create Your Practice" flow, including plan selection and the Paystack payment form.  
  * 4.6 Build the Patient Dashboard, including the "Complete Your Profile" intake form and a view of upcoming/past appointments.  
  * 4.7 Build the complete appointment booking flow: select doctor \-\> view AppointmentCalendar \-\> select slot \-\> confirm booking.  
  * 4.8 Build the Doctor Dashboard: view daily/weekly schedule, and list of pending appointments with actions to approve or cancel.  
  * 4.9 Build the Admin Dashboard: display basic stats (e.g., today's appointments, pending approvals) and user management (invite doctors).  
* 5.0 Infrastructure & Deployment Setup  
  * 5.1 Create a Dockerfile for the backend Node.js application.  
  * 5.2 Create a multi-stage Dockerfile for the frontend React application to build and serve static assets.  
  * 5.3 Create the docker-compose.yml file to orchestrate the backend, frontend (served via nginx), postgres, and redis services.  
  * 5.4 Write the nginx.conf file to act as a reverse proxy, routing /api requests to the backend and serving the frontend for all other requests.  
  * 5.5 Write a detailed, step-by-step deployment guide in the root README.md, including instructions for setting up a VPS, Docker, and obtaining SSL certificates with Let's Encrypt.