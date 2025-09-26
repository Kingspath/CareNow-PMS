# **Product Requirements Document: CareNow MVP (v2.0)**

### **1\. Introduction/Overview**

CareNow is a multi-tenant, SaaS patient management system designed for small to medium-sized healthcare practices in South Africa. The platform aims to solve the core problem of inefficient appointment scheduling and patient management by providing a simple, affordable, and modern digital solution. This MVP will focus on delivering the essential features for clinic onboarding, scheduling, administration, and subscription management, ensuring compliance with POPIA.

### **2\. Goals**

* **Achieve Market Viability:** Implement a monetized, tiered subscription model to ensure the project's financial sustainability and scalability.  
* **Streamline Operations:** Provide a centralized platform for doctors and admins to manage patient appointments, reducing manual administrative work.  
* **Improve Patient Experience:** Offer patients a simple, self-service way to find doctors and book appointments online with a frictionless onboarding process.  
* **Reduce No-Shows:** Decrease the rate of missed appointments through automated email and SMS reminders (available in paid tiers).

### **3\. User Stories**

* **As a Clinic Owner (Admin)**, I want to sign up my practice on a public page and choose a subscription plan so that I can quickly onboard my clinic to the CareNow platform.  
* **As a Doctor**, I want to define my weekly working hours and have the system automatically generate bookable slots, so I don't have to manage every single time slot manually.  
* **As a Patient**, I want to register for an account with minimal effort so I can get started quickly.  
* **As a Patient**, I want to provide my detailed medical and contact information before my first appointment so the clinic is fully prepared for my visit.  
* **As a Growth Tier Clinic Admin**, I want the system to send automated SMS reminders to my patients so that we can reduce costly no-shows.

### **4\. Functional Requirements**

#### **Onboarding and Tenancy**

1. The system must provide a public-facing "Create Your Practice" page where new tenants sign up and select a subscription plan.  
2. Each practice (tenant) operates in isolation, with its own set of doctors, patients, and appointments.  
3. A user with a "Doctor" role can also be assigned an "Admin" role.

#### **Patient Profile & Data Collection**

4. Initial patient registration must be minimal, requiring only **Name, Email/Phone, and Password**.  
5. Before a patient's first appointment booking is confirmed, they must be prompted to complete a detailed intake form.  
6. The detailed patient profile must contain the following fields:  
   * Name & Surname  
   * ID Number (optional)  
   * Medical Aid, Medical Aid Number, Dependent Code (if applicable)  
   * Residential Information (Address)  
   * Occupation  
   * Known Allergies  
   * Emergency Contact Name, Details (Phone/Email), and Relationship.

#### **Scheduling and Appointments**

7. Doctors must be able to define a recurring weekly schedule of their working hours.  
8. The system will automatically generate available appointment slots based on the doctor's schedule and a predefined appointment duration.  
9. A mandatory, configurable buffer time will be automatically added after each appointment.  
10. All patient-booked appointments must be in a "Pending" state until they are manually confirmed by a Doctor or Admin.

#### **Subscription Management & Billing**

11. The system will offer three subscription tiers: **Solo (Free)**, **Growth (Paid)**, and **Pro (Paid)**.  
12. **Paystack integration** will be used to handle all subscription payments.  
13. The application must enforce feature-gating based on the active subscription tier. This includes, but is not limited to:  
    * Limiting the number of practitioners and patients on the Solo tier.  
    * Enabling/disabling SMS notifications.  
    * Providing access to specific analytics and reporting features.

### **5\. Non-Goals (Out of Scope for MVP)**

* **Advanced Billing:** Complex invoicing, pro-rated billing adjustments, and custom enterprise plans are out of scope.  
* **Advanced Scheduling:** Manual one-off time slots, complex recurring appointments, and group bookings will not be supported.  
* **EHR/Medical Records:** The system will not store any clinical patient notes, medical history, or prescriptions.  
* **In-App Messaging:** No real-time chat functionality between patients, doctors, or admins will be included.

### **6\. Design Considerations**

* The UI will be built using **React, Vite, TypeScript, and Tailwind CSS**, with **Shadcn/UI** for the component library.  
* The application must be fully responsive and optimized for modern desktop and mobile browsers.  
* A professional, modern landing page will be created, featuring clear calls-to-action and subtle animations.

### **7\. Technical Considerations**

* **Backend:** Node.js with Express and TypeScript.  
* **Database & Auth:** Supabase will be used for the PostgreSQL database, user authentication (JWT), and object storage.  
* **Error Monitoring:** Sentry will be integrated for real-time error tracking and reporting.  
* **Notifications:** SendGrid for emails and Clickatell for SMS.  
* **Payments:** Paystack will be the designated payment gateway.