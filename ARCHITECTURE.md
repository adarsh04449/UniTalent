# UNI-TALENT Architecture

## 1. Overview

UNI-TALENT is a campus-focused talent and service discovery platform
built for San Diego State University (SDSU) students. The application
allows verified students to create accounts, discover student-provided
services, view portfolios and pricing, and contact other students
directly.

The platform is intentionally designed as a **student talent directory
and discovery system**, not as a full marketplace. UNI-TALENT helps
students find one another, while booking, payment, scheduling, and final
service arrangements happen directly between students.

------------------------------------------------------------------------

## 2. High-Level Architecture

``` text
┌───────────────────────────────────────────────────────────────┐
│                         USER / BROWSER                        │
│                SDSU students using UNI-TALENT                 │
└──────────────────────────────┬────────────────────────────────┘
                               │ HTTPS
                               ▼
┌───────────────────────────────────────────────────────────────┐
│                         VERCEL                                │
│              Hosting, deployment, production                  │
└──────────────────────────────┬────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────┐
│                    NEXT.JS APPLICATION                        │
│                                                               │
│   Pages / Routes       Components        Application Logic    │
│   Authentication       Search/Filters    Profile/Listing UI   │
│                                                               │
│             TypeScript + React + Tailwind CSS                 │
└──────────────────────────────┬────────────────────────────────┘
                               │
                        Supabase Client
                               │
                               ▼
┌───────────────────────────────────────────────────────────────┐
│                         SUPABASE                              │
│                                                               │
│   Authentication        PostgreSQL Database       Storage*    │
│   SDSU Accounts         Profiles / Listings       Media*      │
│                                                               │
│          Security / Row Level Security Policies               │
└───────────────────────────────────────────────────────────────┘

* Used where required by application features.
```

------------------------------------------------------------------------

## 3. Technology Stack

  -----------------------------------------------------------------------
  Layer                   Technology              Responsibility
  ----------------------- ----------------------- -----------------------
  Frontend                Next.js / React         Application pages,
                                                  routing, and user
                                                  interface

  Language                TypeScript              Type-safe application
                                                  development

  Styling                 Tailwind CSS            Responsive styling and
                                                  UI design

  Backend Services        Supabase                Authentication,
                                                  database, and backend
                                                  infrastructure

  Database                PostgreSQL via Supabase Persistent application
                                                  data

  Authentication          Supabase Auth           Account creation,
                                                  login, and email
                                                  verification

  Deployment              Vercel                  Hosting and production
                                                  deployment

  Testing                 Vitest                  Automated application
                                                  testing

  Source Control          GitHub                  Version control and
                                                  collaboration

  CI/CD                   GitHub Actions / Vercel Automated checks and
                                                  deployment workflow
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 4. Repository Structure

``` text
UNI-TALENT/
│
├── .github/
│   └── workflows/          # GitHub Actions / CI workflows
│
├── docs/                   # Project documentation
│
├── public/                 # Static public assets
│
├── src/                    # Main application source code
│   ├── app/                # Next.js routes/pages and layouts
│   ├── components/         # Reusable UI components
│   └── ...                 # Application utilities and logic
│
├── supabase/               # Supabase configuration and database resources
│
├── .env.example            # Required environment variable template
├── .gitignore              # Git exclusions
├── components.json         # UI component configuration
├── eslint.config.mjs       # ESLint configuration
├── next.config.ts          # Next.js configuration
├── package.json            # Dependencies and project scripts
├── postcss.config.mjs      # PostCSS configuration
├── tsconfig.json           # TypeScript configuration
├── vitest.config.mts       # Vitest configuration
└── README.md               # Main project documentation
```

------------------------------------------------------------------------

## 5. Frontend Architecture

The frontend is implemented using **Next.js with React and TypeScript**.

The `src/` directory contains the primary application code. UI
functionality is separated into pages/routes, reusable components, and
supporting application logic.

### Main Frontend Responsibilities

-   Render the UNI-TALENT landing experience.
-   Handle sign-up and sign-in interfaces.
-   Display student profiles and service listings.
-   Support service and skill discovery.
-   Display pricing and portfolio information.
-   Allow users to search and filter available talent.
-   Expose provider contact information to authenticated users.
-   Maintain responsive behavior across desktop and mobile devices.

### Public and Protected Areas

UNI-TALENT separates the application into two conceptual areas:

**Public area**

Accessible without authentication and used to explain the product.

Examples:

``` text
/
 /auth/login
 /auth/sign-up
```

**Protected student area**

Contains student-specific directory functionality and should only be
accessible after authentication and verification.

``` text
Authenticated User
       │
       ▼
Session Validation
       │
       ├── Invalid session ──► Login
       │
       └── Valid session
                │
                ▼
        Verified Student Area
```

------------------------------------------------------------------------

## 6. Authentication Architecture

Authentication is handled through **Supabase Auth**.

UNI-TALENT is designed specifically for the SDSU community, so
registration is based on an SDSU student email address.

### Sign-Up Flow

``` text
Student
   │
   ▼
Create Account
   │
   ├── Full Name
   ├── SDSU Email
   ├── Password
   └── Password Confirmation
   │
   ▼
Validate Registration
   │
   ▼
Supabase Auth
   │
   ▼
Email Confirmation
   │
   ▼
Verified Account
   │
   ▼
UNI-TALENT Directory
```

A confirmation link is sent before the user can enter the private
student directory.

### Sign-In Flow

``` text
Student
   │
   ▼
Enter SDSU Email + Password
   │
   ▼
Supabase Authentication
   │
   ├── Authentication Failed ──► Error / Retry
   │
   └── Authentication Successful
                │
                ▼
           User Session
                │
                ▼
         Protected Directory
```

The authentication layer prevents the private campus directory from
being freely accessible to unauthenticated visitors.

------------------------------------------------------------------------

## 7. Data Architecture

Supabase provides the PostgreSQL database used by the application.

At a conceptual level, the application data is organized around several
core entities.

``` text
AUTH USER
    │
    │ 1 : 1
    ▼
PROFILE
    │
    │ 1 : many
    ▼
SERVICE LISTING
    │
    ├────────► SKILL / CATEGORY
    │
    └────────► PORTFOLIO CONTENT
```

### User

Authentication-level information is managed by Supabase Auth.

Typical information includes:

-   User ID
-   Email
-   Authentication credentials
-   Email verification state
-   Account timestamps

### Profile

Stores the public-facing information associated with a student.

Conceptually:

``` text
Profile
├── user_id
├── full_name
├── bio
├── contact_information
├── profile_image
└── created_at
```

### Service Listing

Represents a service or skill offered by a student.

Conceptually:

``` text
Service
├── id
├── provider_id
├── title
├── description
├── category
├── price
├── pricing_type
├── portfolio/media
└── created_at
```

Exact database columns should remain defined by the Supabase
migrations/schema rather than duplicated as a second source of truth in
this document.

------------------------------------------------------------------------

## 8. Application Data Flow

A typical read operation follows this path:

``` text
Student Browser
      │
      ▼
Next.js Page / Component
      │
      ▼
Supabase Client
      │
      ▼
Authentication / Authorization
      │
      ▼
PostgreSQL
      │
      ▼
Requested Records
      │
      ▼
Next.js UI
      │
      ▼
Rendered Student Directory
```

A typical write operation follows:

``` text
Student submits data
        │
        ▼
Client-side validation
        │
        ▼
Authenticated Supabase request
        │
        ▼
Authorization / RLS checks
        │
        ▼
PostgreSQL mutation
        │
        ▼
Updated application state
        │
        ▼
Updated UI
```

------------------------------------------------------------------------

## 9. Security Model

Security is especially important because UNI-TALENT contains a private
university community directory.

The architecture uses multiple layers of protection.

### Authentication

Supabase Auth establishes the identity of each signed-in user.

### University Email Verification

Account registration is restricted to the intended SDSU community, with
email confirmation used before access to the private directory.

### Authorization

Authenticated access alone should not automatically grant permission to
modify arbitrary data.

Database authorization should enforce rules such as:

-   Students can edit their own profiles.
-   Students can create listings associated with their own accounts.
-   Students can update or delete their own listings.
-   One student cannot modify another student's profile or listings.
-   Protected student information is not exposed to unauthenticated
    users.

### Row Level Security

Supabase Row Level Security (RLS) should provide database-level
enforcement for user-owned data.

This ensures security does not depend only on hiding controls in the
frontend.

### Environment Variables

Sensitive configuration is stored through environment variables and is
not committed directly to the repository.

The `.env.example` file documents required variables without exposing
production secrets.

------------------------------------------------------------------------

## 10. Discovery Architecture

UNI-TALENT's central product feature is talent discovery.

The basic discovery flow is:

``` text
Student enters directory
        │
        ▼
Browse / Search Services
        │
        ├── Skill
        ├── Category
        └── Price
        │
        ▼
View Matching Listings
        │
        ▼
Open Student / Service
        │
        ├── Description
        ├── Pricing
        ├── Portfolio
        └── Contact Method
        │
        ▼
Contact Provider Directly
```

This architecture keeps discovery inside UNI-TALENT while allowing
students to arrange the actual work independently.

------------------------------------------------------------------------

## 11. Marketplace Boundary

UNI-TALENT currently operates as a **discovery and connection
platform**, rather than a transaction-processing marketplace.

The application is responsible for:

-   Student verification
-   Profiles
-   Service listings
-   Skill discovery
-   Search and filtering
-   Portfolio presentation
-   Pricing visibility
-   Contact discovery

The application does **not** need to own:

-   Payment processing
-   Payment custody
-   Service fulfillment
-   Appointment booking
-   Contract management
-   Platform transaction fees

The boundary can be represented as:

``` text
             UNI-TALENT RESPONSIBILITY

Discover ──► Evaluate ──► View Portfolio ──► Contact
                                                   │
                                                   ▼
                         ─────────────────────────────
                          Platform boundary
                         ─────────────────────────────
                                                   │
                                                   ▼
                           Student-to-Student
                         Scheduling / Payment /
                           Service Fulfillment
```

This separation reduces application complexity and keeps the MVP focused
on the primary problem: making campus talent discoverable.

------------------------------------------------------------------------

## 12. Deployment Architecture

The production application is deployed through **Vercel**.

``` text
Developer
    │
    ▼
Git Repository
    │
    ▼
GitHub
    │
    ├── CI / automated checks
    │
    ▼
Vercel Build
    │
    ├── Install dependencies
    ├── Build Next.js application
    ├── Apply production configuration
    └── Deploy
    │
    ▼
https://uni-talent.vercel.app
    │
    ▼
Supabase Backend
```

Vercel hosts the application while Supabase provides persistent backend
services.

This separates the web application deployment layer from the database
and authentication layer.

------------------------------------------------------------------------

## 13. CI/CD

The `.github/workflows/` directory contains repository automation.

The intended development pipeline is:

``` text
Code Change
    │
    ▼
Git Commit / Push
    │
    ▼
GitHub Repository
    │
    ▼
Automated Checks
    │
    ▼
Production Build
    │
    ▼
Vercel Deployment
```

Automated checks help prevent broken code from reaching production.

------------------------------------------------------------------------

## 14. Testing

The repository includes a `vitest.config.mts` configuration, indicating
**Vitest** is used for automated testing.

Testing should focus on important application behavior including:

-   Authentication validation
-   SDSU email requirements
-   Service/listing logic
-   Search and filtering
-   User-owned data operations
-   Reusable UI components
-   Utility functions

Database authorization and RLS policies should also be tested
independently because frontend tests alone cannot guarantee database
security.

------------------------------------------------------------------------

## 15. Design Principles

UNI-TALENT follows several architectural principles.

### Campus First

The platform is designed around a defined university community instead
of an unrestricted public marketplace.

### Portfolio First

Students can evaluate actual work, skills, pricing, and service
information before making contact.

### Direct Connection

UNI-TALENT facilitates discovery rather than inserting itself into every
stage of the transaction.

### Clear Separation of Concerns

``` text
Next.js       → Interface and application behavior
Supabase Auth → Identity and authentication
PostgreSQL    → Persistent application data
Supabase RLS  → Data authorization
Vercel        → Hosting and deployment
GitHub        → Version control and CI workflow
```

### MVP Focus

The architecture avoids unnecessary transaction infrastructure such as
payment processing and booking systems until those capabilities are
justified by product requirements.

------------------------------------------------------------------------

## 16. Future Architecture Considerations

The current architecture can be extended as UNI-TALENT grows.

Potential future capabilities include:

-   Reviews and ratings
-   Favorites / saved providers
-   Advanced search
-   Availability indicators
-   In-app messaging
-   Notifications
-   Reporting and moderation
-   Admin dashboards
-   Additional university support
-   Analytics
-   Recommendation systems

These should be added incrementally without changing the core separation
between the Next.js application and Supabase backend services.

------------------------------------------------------------------------

## 17. Architecture Summary

``` text
                       ┌──────────────────┐
                       │   SDSU Student   │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │      Vercel      │
                       └────────┬─────────┘
                                │
                                ▼
                 ┌──────────────────────────┐
                 │      Next.js / React     │
                 │        TypeScript        │
                 │       Tailwind CSS       │
                 └────────────┬─────────────┘
                              │
                              ▼
                 ┌──────────────────────────┐
                 │         Supabase         │
                 ├──────────────────────────┤
                 │ Authentication           │
                 │ PostgreSQL Database      │
                 │ Row Level Security       │
                 │ Storage (where required) │
                 └──────────────────────────┘
```

UNI-TALENT uses a modern serverless web architecture in which **Next.js
provides the application experience, Supabase provides backend data and
authentication services, and Vercel provides the production deployment
environment**.

The architecture is intentionally centered on the application's primary
purpose: helping verified SDSU students discover campus talent, evaluate
student-provided services, and connect directly.
