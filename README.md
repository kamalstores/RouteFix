# RouteFix - Comprehensive Technical Analysis & Interview Guide

This document is a deep, extensive technical analysis of the **RouteFix** repository. It is designed to take you from zero knowledge to a complete, interview-ready understanding of the project's architecture, design decisions, and implementation details.

---

## 1. PROJECT OVERVIEW

**What problem does this project solve?**
Managing physical store maintenance, equipment repairs, and IT issues is chaotic. Stores usually rely on phone calls or messy email chains to request help from technicians. RouteFix streamlines this by providing a unified ticketing system that automatically classifies issues, determines priority, calculates Service Level Agreements (SLAs), and routes the ticket to the best available service provider using Artificial Intelligence.

**Why does this problem exist?**
Large retail chains (like Walmart) have thousands of physical assets and complex infrastructures. Traditional ticketing systems are manual, requiring a human dispatcher to read a ticket, figure out what skills are needed, and manually assign a technician, leading to delays and human error.

**Who would use this project?**
1. **Store Managers/Employees:** To report issues.
2. **Technicians (Service Providers):** To receive, accept, and resolve work orders.
3. **System Moderators/Admins:** To approve new users and oversee escalated tickets.

**What is the main goal of the project?**
To eliminate human bottleneck in issue dispatching by automating classification and routing using AI, ensuring faster response times and optimized technician utilization.

**What are the major features?**
* Role-based access control (Store, Provider, Moderator, Admin).
* AI-powered ticket classification and SLA deadline calculation (using LangChain/Google GenAI).
* Intelligent routing system that matches ticket requirements with technician skills and availability.
* Comprehensive dashboard for technicians to manage their queue.
* Escalation monitoring for missed SLAs.

**What makes this project useful?**
It bridges the gap between physical retail operations and modern AI automation. It reduces downtime for store equipment.

**What would happen if this project didn't exist?**
Stores would suffer longer equipment downtimes, leading to lost revenue (e.g., broken POS system or refrigerators), and dispatchers would be overwhelmed with manual routing.

### Elevator Pitches
* **One sentence:** An AI-powered ticketing and dispatch system that automates the classification and routing of maintenance issues for retail stores.
* **30 seconds:** RouteFix is a B2B facility management platform. When a store has a problem, like a broken fridge, they submit a ticket. Our system uses an AI agent to instantly read the description, classify it as an HVAC issue, assign a high priority, calculate a deadline, and automatically dispatch it to the nearest available technician with the right skills.
* **2 minutes:** (Expand on the 30-second pitch) We built this using Next.js, Prisma, and PostgreSQL. The core innovation is our AI Orchestrator, powered by LangChain and Google GenAI. Instead of a human reading tickets, four distinct AI agents handle classification, SLA calculation, availability matching, and routing. This completely automates the dispatcher role. For the frontend, we use React with Radix UI and Tailwind for a clean, accessible interface. NextAuth handles secure, role-based authentication so stores, technicians, and moderators all have distinct, protected workflows.
* **5 minutes:** (Go into database schema, API design, and edge cases, mirroring the sections below).

**Real-world analogy:** 
Imagine Uber, but instead of routing a car to a passenger, it routes a specialized plumber to a broken pipe in a grocery store, using AI to make sure the plumber actually knows how to fix that specific type of pipe.

---

## 2. TECH STACK — COMPLETE BREAKDOWN

| Technology | Where Used | Why Used | Alternative | Why This One Was Chosen |
| ---------- | ---------- | -------- | ----------- | ----------------------- |
| **Next.js (App Router)** | Fullstack Framework | Provides seamless frontend/backend integration, SSR, and API routes. | React + Express | Faster development speed, built-in routing, and optimal performance via SSR/RSC. |
| **TypeScript** | Entire codebase | Catches errors at compile-time, provides excellent IDE autocompletion. | JavaScript | Essential for a complex data model to prevent runtime type errors. |
| **PostgreSQL** | Database | Relational data structure fits the highly relational nature of the app (Users, Stores, Tickets). | MongoDB | The app requires strict schemas, ACID transactions, and complex joins (e.g., finding a Provider with specific skills for a Ticket). |
| **Prisma** | Backend ORM | Type-safe database queries that sync perfectly with TypeScript. | TypeORM / Drizzle | Developer experience is unmatched; auto-generated types prevent query errors. |
| **NextAuth.js** | Authentication | Handles secure login, sessions, and JWT tokens natively in Next.js. | Clerk / Firebase Auth | Keeps authentication strictly within the app's control without relying on expensive third-party paid services. |
| **Tailwind CSS** | Frontend Styling | Utility-first styling for rapid UI development. | Styled Components / SCSS | No context switching between CSS files; smaller bundle sizes in production. |
| **Radix UI** | Frontend UI Primitives | Provides accessible, unstyled UI components (Dialogs, Menus). | Material UI / Chakra UI | Complete design freedom while guaranteeing web accessibility (a11y) standards. |
| **LangChain & Google GenAI** | AI Orchestration (`lib/ai`) | To build the AI agents that classify and route tickets. | OpenAI API directly | LangChain provides structure for multi-agent workflows (Orchestrator pattern). |
| **Docker** | Development | Containerizes the PostgreSQL database for easy local setup. | Local Postgres Install | Ensures every developer has the exact same database environment without system pollution. |

---

## 3. REPOSITORY STRUCTURE

```text
RouteFix/
├── app/                  # Next.js App Router (Frontend Pages & Backend APIs)
│   ├── api/              # Backend REST API routes
│   │   ├── auth/         # NextAuth configuration routes
│   │   ├── tickets/      # Ticket management APIs
│   │   └── ...
│   ├── auth/             # Frontend login/register pages
│   ├── admin/            # Admin dashboard UI
│   ├── store/            # Store dashboard UI
│   └── technician/       # Technician dashboard UI
├── components/           # Reusable React UI components (Radix/Tailwind)
├── lib/                  # Core Business Logic & Utilities
│   ├── ai/               # AI Agents (Classification, Routing, Orchestrator)
│   ├── auth/             # Role-based Access Control (RBAC) logic
│   └── prisma.ts         # Database client singleton
├── prisma/               # Database Schema
│   └── schema.prisma     # Defines Tables, Relations, and Enums
└── docker-compose.yml    # Database containerization
```

**Why this structure?**
It follows standard Next.js convention. `app` handles routing (UI and API), `components` holds view-layer building blocks, and `lib` isolates business logic from the web framework, making the code testable and modular.

---

## 4. ARCHITECTURE

**Architecture Pattern:** Modular Monolith (Client-Server in one repository).
Because it uses Next.js, the frontend (React Server Components/Client Components) and the backend (API Routes) live in the same repository and run on the same server, but are logically separated.

**Data Flow Diagram:**
```text
User (Store/Tech)
       ↓ (HTTP Request)
Next.js Frontend (React)
       ↓ (Fetch API)
Next.js API Route (app/api/...)
       ↓ (Function Call)
AI Orchestrator (lib/ai/orchestrator.ts) ←→ LangChain / Google GenAI
       ↓ (Function Call)
Prisma ORM (lib/prisma.ts)
       ↓ (SQL Query)
PostgreSQL Database
```

**Why is this appropriate?**
A microservices architecture would be massive overkill for this stage. A modular monolith provides the fastest iteration speed while keeping the codebase cohesive. The heavy lifting (AI logic) is modularized in `lib/ai`, so it could be extracted to a Python microservice later if needed.

---

## 5. COMPLETE APPLICATION FLOW: Creating a Ticket

**Use Case:** A store manager reports a broken refrigerator.

1. **Frontend (`app/store/page.tsx`):** User fills out a form and clicks "Submit Ticket".
2. **API Route (`app/api/tickets/route.ts`):** Receives the POST request. Validates the input.
3. **Service Layer (`lib/ai/orchestrator.ts`):** 
   * **Classify:** Calls `classificationAgent.classify()`. The AI determines this is `Facilities_HVAC`, `High` priority.
   * **SLA:** Calls `escalationAgent.calculateSLADeadline()`. Calculates a 4-hour deadline.
   * **Database Write:** Uses Prisma to `INSERT` the new ticket into the `Ticket` table.
   * **Availability Check:** Calls `availabilityAgent.getAvailableProviders()` to find HVAC technicians near the store with `current_load < capacity`.
   * **Routing:** Calls `routingAgent.routeTicket()` to assign it to the absolute best match.
4. **API Route:** Returns a `200 OK` with the created ticket and assigned provider ID.
5. **Frontend:** Updates the React state to show the ticket in the "Active Tickets" list.

---

## 6. CODE WALKTHROUGH: `lib/ai/orchestrator.ts`

**Purpose:** This is the "brain" of the application. It orchestrates multiple AI agents to process a ticket completely hands-free.

**Important Logic (`processNewTicket` function):**
1. **Input:** `ticketData` (description, store_id, etc.).
2. **Classification:** Uses a language model to extract the category and priority from plain text.
3. **Database Transaction:** Creates the ticket.
4. **Skill Matching (`getRequiredSkills`):** Maps AI categories (e.g., "Facilities_Electrical") to actual technician skills (e.g., ["Electrical"]).
5. **Rejection Handling (`handleTicketRejection`):** If a tech rejects a ticket, this function removes them from the assignment, logs a "Remark" (audit trail), and recursively re-routes the ticket to the *next* best provider. If no providers are left, it marks the ticket as `ESCALATED`.

**Why written this way?**
It strictly separates concerns. The orchestrator doesn't know *how* to classify text; it delegates that to `classificationAgent`. It just controls the flow of data between the AI models and the PostgreSQL database.

---

## 7. DATABASE DEEP DIVE (`prisma/schema.prisma`)

**Technology:** PostgreSQL managed by Prisma ORM.

**Core Tables & Relationships:**
1. **`User`**: Contains all credentials. Has an Enum `role` (STORE_REGISTER, SERVICE_PROVIDER, ADMIN, MODERATOR).
2. **`Store`**: Physical locations. Linked to `User` (1-to-many: a store has many users).
3. **`ServiceProvider`**: Technician companies. Contains `skills` (Array of strings) and `current_load`.
4. **`Ticket`**: The core entity.
   * Belongs to a `Store`.
   * Belongs to a `ServiceProvider` (assigned_provider).
   * Has many `TicketAssignment` (history of who it was offered to).
   * Has many `Remark` (audit trail/comments).

**Why PostgreSQL?**
The relationships are complex. A Ticket is linked to a Store, a Reporter (User), an Assigned Provider (ServiceProvider), and has an array of Assignments and Escalations. Querying this in a NoSQL database (like MongoDB) would require multiple round trips or massive document duplication. SQL handles this perfectly with JOINs (handled via Prisma's `include` feature).

**Data Lifecycle (Ticket Acceptance):**
Technician clicks "Accept" → API calls `orchestrator.handleTicketAcceptance` → Prisma updates `TicketAssignment.status` to 'ACCEPTED' → Prisma updates `Ticket.status` to 'IN_PROGRESS' → Returns success.

---

## 8. API DEEP DIVE

All APIs are Next.js App Router Route Handlers (`app/api/.../route.ts`).

| Method | Endpoint | Purpose | Authentication |
| ------ | -------- | ------- | -------------- |
| POST | `/api/auth/register` | Creates a new User. | Public |
| POST | `/api/tickets` | Store creates a ticket; triggers AI. | Required (Store Role) |
| GET | `/api/tickets` | Fetches tickets for the logged-in user. | Required (Any Role) |
| PUT | `/api/tickets/[id]/accept` | Tech accepts a routed ticket. | Required (Provider Role) |

**REST Principles:** The app follows standard RESTful patterns. Resources are identified by URLs (`/api/tickets`), and HTTP verbs dictate the action (POST for create, GET for read, PUT for update).

---

## 9. AUTHENTICATION & AUTHORIZATION

**Authentication (Who are you?):** Handled by `NextAuth.js`.
* Passwords are hashed using `bcryptjs` before being stored in the database.
* Upon login, NextAuth verifies the hash and issues a secure, HTTP-only JWT (JSON Web Token) cookie.

**Authorization (What can you do?):** Handled by custom RBAC (Role-Based Access Control) in `lib/auth/config.ts` and `lib/auth/rbac.ts`.
* **Middleware:** Next.js middleware intercepts requests. If a user tries to access `/admin/...` but their JWT role is `STORE_REGISTER`, they are redirected away.
* **API Protection:** Every secure API route first checks the NextAuth session to ensure the user has the right permissions to perform the action.

---

## 10. FRONTEND DEEP DIVE

**Architecture:** Next.js App Router (React Server Components + Client Components).
* **Styling:** Tailwind CSS for layout and design.
* **Components:** Radix UI primitives wrapped in a custom design system (likely based on `shadcn/ui` given the dependencies like `class-variance-authority`, `clsx`, `tailwind-merge`).

**State Management:**
Because this is a standard Next.js application, much of the "state" is managed via URL routing and server-side fetching. For interactive client state (like forms), React Hooks (`useState`, `useForm` from `react-hook-form` + `zod` for validation) are used.

**UI Flow:**
User clicks "Create Ticket" → `react-hook-form` validates inputs → `fetch()` pushes to `/api/tickets` → Upon success, `useRouter().refresh()` is called to invalidate the cache and fetch the newly updated ticket list from the server.

---

## 11. DESIGN DECISIONS

* **Why Next.js?** Allows a single developer or small team to build a full-stack application without maintaining two separate repositories (a React frontend and a Node.js backend).
* **Why Prisma?** Prisma's schema file acts as a single source of truth. When the schema changes, Prisma regenerates TypeScript types. This means if you change a database column, your frontend code will instantly throw a compile error if it uses the old name. This prevents massive amounts of bugs.
* **Why LangChain?** Instead of just sending a prompt to OpenAI/Google, LangChain allows the app to define a strict "Agent" workflow. The ticket isn't just processed by one prompt; it goes through a Classification Agent, then an Availability Agent, then a Routing Agent. This Pipeline architecture makes the AI predictable and debuggable.

---

## 12. PROBLEMS & CHALLENGES

### Challenge 1: Hallucinations in AI Routing
* **Problem:** AI models can hallucinate or format data incorrectly (e.g., returning plain text instead of JSON).
* **Solution:** The Orchestrator uses structured prompting and separates tasks into distinct agents. By keeping the AI focused on one tiny task at a time (just classify, don't route), error rates drop significantly. (🟢 Strong inference based on the presence of multiple agent files in `lib/ai/agents`).

### Challenge 2: Concurrency & Race Conditions
* **Problem:** What if the AI routes a ticket to Technician A, but 1 second later, Technician A accepts a different ticket and their `current_load` maxes out?
* **Solution:** The database uses transactional updates. When a tech accepts a ticket, the backend must verify their load hasn't exceeded capacity *at the exact moment of database insertion*. 

---

## 13. SECURITY ANALYSIS

* ✅ **Secure:** Passwords are hashed (`bcryptjs`). Sessions are managed securely via NextAuth HTTP-only cookies.
* ✅ **Secure:** ORM (Prisma) automatically escapes queries, preventing SQL Injection.
* 🟡 **Risky:** Route protection must be meticulously applied. If an API route forgets to check `session.user.role`, a Store user could theoretically call a Moderator API endpoint.
* 🔵 **Recommendation:** Implement strict Rate Limiting on the `/api/tickets` creation route to prevent malicious users from spamming the expensive AI endpoints (Google GenAI) and racking up a massive API bill.

---

## 14. PERFORMANCE & SCALABILITY

**Current Bottleneck:** The AI Orchestrator.
When a ticket is submitted, the user has to wait for the API to call the AI model, wait for the response, query the DB for providers, call the AI model again for routing, and then write to the DB. This could take 3-5 seconds, which is a slow HTTP response.

**Scaling Solution (If 100,000 users):**
We would need to decouple the AI logic from the HTTP request cycle using an asynchronous **Message Queue** (like Redis + BullMQ or AWS SQS).
1. User submits ticket → API saves ticket as "PENDING_AI" → API returns `200 OK` instantly.
2. Background Worker picks up the ticket from the queue → Runs the `AIOrchestrator` → Updates ticket to "ASSIGNED".
3. Frontend uses WebSockets or polling to update the UI when the status changes.

---

## 15. INTERVIEW PREPARATION

### Intermediate: Architecture
**Q: Explain the architecture of the RouteFix backend.**
> **Ideal Answer:** "RouteFix uses a modular monolithic architecture built on Next.js API Routes. It follows a layered pattern. The HTTP routes act as Controllers. They immediately pass data to the Service Layer—specifically our AI Orchestrator. The Orchestrator handles business logic, interacting with LangChain agents for decision making, and finally uses Prisma as the Data Access Layer to persist state in PostgreSQL."

### Advanced: System Design
**Q: How does the system ensure that a technician isn't assigned a ticket they don't have the skills for?**
> **Ideal Answer:** "It's a two-step process. First, the Classification AI agent reads the ticket and maps it to a standardized category, like 'Facilities_HVAC'. We have a strict mapping function in our Orchestrator that translates that category into required technician skills. Next, we query the PostgreSQL database for providers where their `skills` array contains the required skills, and their `current_load` is less than their capacity. Only those filtered providers are passed to the Routing AI to make the final assignment decision."

### Expert: Trade-offs
**Q: You chose to run complex AI orchestration synchronously in the API route. What is the trade-off here?**
> **Ideal Answer:** "The benefit is architectural simplicity—we don't need to manage a separate background worker infrastructure (like Redis/Celery) or deal with complex WebSocket updates on the frontend. The major trade-off is latency and reliability. The user has to wait several seconds for the AI to process before the UI updates. Furthermore, if the AI API times out, the entire request fails. If I were to scale this, moving the orchestrator to an asynchronous event-driven queue would be my first priority."

---

## 16. "EXPLAIN LIKE I'M IN AN INTERVIEW" (2-Minute Script)

"The main problem we were trying to solve with RouteFix was the severe bottleneck in facility management dispatching. Stores were submitting tickets, and human dispatchers were getting overwhelmed trying to figure out who to send the ticket to based on skills and location.

We built a solution using a modern Next.js and PostgreSQL stack. The core of the system is what we call the AI Orchestrator. When a store submits a ticket, it doesn't just go into a dumb queue. It hits our Next.js API, which passes it to our Orchestrator. The Orchestrator uses LangChain and Google's GenAI to read the plain English description, classify the issue, and calculate a strict SLA deadline. 

Then, it queries our PostgreSQL database via Prisma to find all technicians within a specific radius who have the exact skills required and aren't overloaded with work. Finally, it uses a Routing AI agent to pick the absolute best technician and assigns the ticket automatically. 

We chose this architecture because it allowed us to completely automate a complex human workflow. One of the challenges was ensuring the AI didn't hallucinate assignments, which is why we strictly query the database for eligible technicians *first*, and only allow the AI to choose from that pre-validated list. It's a robust, intelligent system designed to scale physical retail operations."

---

## 17. FINAL KNOWLEDGE MAP

```text
RouteFix
├── Core Problem: Manual ticket dispatching is slow and error-prone.
├── Key Innovation: AI Orchestrator (lib/ai/orchestrator.ts)
│   ├── 1. Classifies text via LLM
│   ├── 2. Queries DB for eligible techs via Prisma
│   └── 3. Routes ticket automatically
├── Tech Stack
│   ├── Next.js (Fullstack Framework)
│   ├── PostgreSQL (Relational Data)
│   ├── Prisma (Type-safe ORM)
│   ├── NextAuth (Authentication)
│   └── LangChain (AI Workflow)
└── Scaling Bottleneck: Synchronous AI API calls (Needs a Message Queue).
```
