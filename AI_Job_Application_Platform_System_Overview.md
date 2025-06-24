# Staged, Iterative Architecture for AI Job App Automator

This document outlines a staged, iterative architecture for each core system dimension—**GraphQL schema**, **ATS connector/plugin extensibility**, and **UI/stepper flow**—designed for incremental, testable development. Each stage builds upon the last, ensuring clean evolution, extensibility, and maintainability.

---

## 1️⃣ Backend Contract: GraphQL Schema (Staged Evolution)

### **Stage 1 – Core Entities & Simple Flows**

- Define core types: `User`, `JobApplication`, `SessionStep`, `JobSpec`
- Basic mutations for starting an application, fetching state, and approving steps.

```graphql
type User { id: ID! email: String! }
type JobSpec { id: ID! url: String! title: String! company: String! }
type JobApplication { id: ID! user: User! jobSpec: JobSpec! status: String! steps: [SessionStep!]! }
type SessionStep { id: ID! type: String! data: JSON! aiSuggestion: JSON status: String! requiresUserAction: Boolean! }

type Mutation {
  startApplication(jobUrl: String!): JobApplication!
  approveStep(applicationId: ID!, stepId: ID!, data: JSON): SessionStep!
}
type Query {
  application(id: ID!): JobApplication
}
```

---

### **Stage 2 – Workflow State & Interruptions**

- Add pause, resume, and stop mutations.
- Store workflow status and checkpoint logic.

```graphql
type Mutation {
  pauseApplication(applicationId: ID!): Boolean!
  resumeApplication(applicationId: ID!): SessionStep!
  stopApplication(applicationId: ID!): Boolean!
}
```

---

### **Stage 3 – AI Module Support & Plugin-Driven Fields**

- Enable attaching AI-generated suggestions to steps and plug-in registered fields (for extensibility).
- Support for registering plugin workflows per job board/ATS.

```graphql
type AIModuleSuggestion { label: String! value: String! confidence: Float }
type SessionStep {
  ...
  aiSuggestions: [AIModuleSuggestion!]
  pluginId: String
}
type Query {
  supportedATS: [String!]!
}
```

---

### **Stage 4 – Audit, Attachments, and Analytics**

- Track all user/AI actions and step transitions.
- Add support for resume/cover letter uploads.

```graphql
type ApplicationAuditEvent { timestamp: DateTime! action: String! details: JSON }
type JobApplication {
  ...
  auditLog: [ApplicationAuditEvent!]!
  attachments: [File!]
}
type File { id: ID! name: String! url: String! type: String! }
```

---

### **Stage 5 – Multi-user, Roles, and Notifications**

- Add organization support, real-time notifications (e.g., for manual action required), and assignment of applications.

```graphql
type Organization { id: ID! name: String! users: [User!]! }
type Notification { id: ID! type: String! message: String! seen: Boolean! }
type Query { notifications(userId: ID!): [Notification!]! }
```

---

## 2️⃣ Extensibility: ATS Connector & Plugin System (Staged Approach)

### **Stage 1 – Static ATS Connector**

- Implement ATS connectors as code modules.
- Registry maps URLs to connector handlers.

```typescript
// connector-registry.ts
export interface ATSConnector { id: string; supports(url: string): boolean; ... }
const connectors = [LinkedInConnector, GreenhouseConnector, ...];
export function getConnector(url: string) { ... }
```

---

### **Stage 2 – Pluggable Module System**

- Allow dynamic plugin discovery/registration.
- Plugins export lifecycle hooks: e.g. `onJobSpecExtract`, `onStepRender`, `detectAntiBot`.

```typescript
export interface PluginModule {
  id: string
  register(app: Application): void
}
export function registerPlugin(plugin: PluginModule) { ... }
```

---

### **Stage 3 – User/Org-Scoped Plugins**

- Plugins can be enabled per user/organization.
- Plugin config and per-user settings persisted in DB.

```typescript
type PluginConfig { id: string; enabled: boolean; config: JSON }
type User { ... plugins: [PluginConfig!] }
```

---

### **Stage 4 – Safe Sandbox & Versioning**

- Plugins run in a sandbox (e.g., vm2 or subprocess).
- Support plugin versioning and dependency locking.

---

### **Stage 5 – UI Extension Points**

- Plugins can inject UI components/steps.
- E.g., custom stepper panes, AI feedback UI, or field mappers.

---

## 3️⃣ UI/Stepper Flow: Developer Experience (Staged Approach)

### **Stage 1 – Linear Stepper**

- Minimal UI: file upload, job link input, sequential approval steps.
- Manual advance: [Next], [Pause], [Stop] at each stage.

---

### **Stage 2 – Stepper + Inline AI Suggestions**

- Preview form with AI-filled fields; editable by user before “Next.”
- Explicit manual action for CAPTCHA/2FA or unknown fields.

---

### **Stage 3 – Pausable/Resumable State**

- UI state persists across reloads.
- Stepper reflects paused, stopped, in-progress status, and allows resume from checkpoint.

---

### **Stage 4 – Pluggable Step Components**

- Steps dynamically rendered based on ATS connector/plugin (e.g., “GreenhouseResumeUploadStep”).
- Plugins can add/override UI at certain steps.

---

### **Stage 5 – History, Logs, and Notifications**

- Application timeline view (step-by-step, with audit log).
- Notification panel: alerts for manual actions required or completed steps.

---

### **Summary Table: Staged Build-Up**

| Stage | GQL Schema Focus           | ATS Extensibility Focus         | UI/Stepper Focus                |
| ----- | -------------------------- | ------------------------------- | ------------------------------- |
| 1     | Core entities, basic flows | Static code modules, registry   | Linear stepper, core actions    |
| 2     | Workflow state, interrupts | Pluggable modules, discovery    | AI preview, user field editing  |
| 3     | AI, plugin-driven fields   | Per-user/org plugin config      | Pausable/resumable, checkpoints |
| 4     | Audit, uploads, analytics  | Sandboxed plugins, versioning   | UI extension points, plugin UI  |
| 5     | Multi-user, notifications  | Full plugin lifecycle, advanced | Timeline, logs, notifications   |

---

## AI Codex Concise YAML Prompt

```yaml
context-ai-job-app-automator:
  phases:
    - id: backend-gql
      name: Backend Contract – GraphQL Schema (Node + Postgres)
      stages:
        - id: stage1
          name: Core Entities & Simple Flows
          objective: >
            Build a Node.js backend using Apollo Server and PostgreSQL (Prisma). Implement GraphQL API for job applications with human-in-the-loop workflow. Include entities: User, JobApplication, JobSpec, SessionStep. Implement user registration, application creation, workflow state fetching, and step approval. Include Jest-based E2E tests.
          stack:
            - Node.js (TypeScript)
            - Apollo Server
            - PostgreSQL
            - Prisma ORM
            - Jest (E2E)
            - ts-node-dev
          gql_types: |
            type User {
              id: ID!
              email: String!
            }
            type JobSpec {
              id: ID!
              url: String!
              title: String!
              company: String!
            }
            type JobApplication {
              id: ID!
              user: User!
              jobSpec: JobSpec!
              status: String!
              steps: [SessionStep!]!
            }
            type SessionStep {
              id: ID!
              type: String!
              data: JSON!
              aiSuggestion: JSON
              status: String!
              requiresUserAction: Boolean!
            }
            type Query {
              application(id: ID!): JobApplication
            }
            type Mutation {
              startApplication(jobUrl: String!): JobApplication!
              approveStep(applicationId: ID!, stepId: ID!, data: JSON): SessionStep!
            }
          features:
            - User registration/login (email only, for demo)
            - startApplication: creates JobApplication, extracts job spec (stub), initializes first SessionStep
            - approveStep: records user approval and advances workflow
            - application query: fetches application state and steps
          tests:
            - Jest + Apollo Test Client
            - Covers: user registration, application start, workflow state, step approval, state transitions
          deliverables:
            - Complete TypeScript codebase (strongly typed)
            - All code and tests in a clear structure
            - .env.example for DB
            - All E2E tests passing
            - README with setup and test instructions
          next:
            - Await further instruction for pause/resume, AI modules, audit logging

    - id: ats-plugin
      name: ATS Connector & Plugin System
      stages:
        - id: stage1
          name: Static ATS Connector
          objective: >
            Implement a pluggable architecture for ATS connectors (e.g., LinkedIn, Greenhouse) with an in-memory registry. Each connector supports URL detection, job spec extraction, and workflow step generation. Include Jest-based E2E tests.
          stack:
            - TypeScript
            - Node.js
            - Jest
          interface: |
            export interface ATSConnector {
              id: string;
              supports(url: string): boolean;
              getJobSpec(url: string): Promise<JobSpec>;
              getFormSteps(url: string): Promise<FormStep[]>;
            }
          features:
            - In-memory registry: registerConnector, getConnectorForUrl
            - At least two connectors (LinkedIn, Greenhouse)
            - Stubbed getJobSpec for mock data
          tests:
            - Selects correct connector for URL
            - getJobSpec returns correct stub data
          deliverables:
            - All code in a directory with interfaces, registry, connectors
            - All tests in __tests__ directory
          next:
            - Await instruction for dynamic plugin loading, sandboxing, user-scoped plugin config

    - id: ui-stepper
      name: UI Stepper Flow
      stages:
        - id: stage1
          name: Linear Stepper UI
          objective: >
            Build a React (TypeScript) app with a stepper component for the job application workflow. Each step displays job data, AI suggestions (mocked), and requires user action to proceed. E2E tested with Cypress.
          stack:
            - React (TypeScript)
            - Vite or Next.js
            - Tailwind CSS
            - Cypress
          features:
            - Step 1: Resume/profile upload (accept PDF, DOCX, TXT)
            - Step 2: Input job URL
            - Step 3: Preview job spec & AI cover letter (mock)
            - Step 4: User approval/edit, advance step
            - Step 5: Completion screen
            - Controls: Next, Back, Pause, Stop at every step
            - State in React context or localStorage
          tests:
            - Cypress E2E: upload, enter job URL, navigate steps, edit/approve fields, pause/stop/resume
          deliverables:
            - Full React project structure
            - Cypress tests in cypress/e2e/
            - README with run/test instructions
          next:
            - Await instruction for inline AI suggestions, resumable state, plugin-driven step rendering

  acceptance_criteria:
    - All code must be strongly typed (TypeScript strict)
    - E2E tests must fully cover primary flows
    - No code comments except where strictly needed
    - Output as self-contained repo or monorepo per phase
    - All config/examples included for reproducibility
    - Clean, idiomatic modern code, minimal dependencies
    - All future stages must build on previous without breaking compatibility

  guidance:
    - Implement and fully test each phase/stage before advancing to the next
    - Confirm E2E tests passing before requesting Stage 2 of any phase
    - Code should support easy plug-in/extension in subsequent stages
```

# 🚀 AI-Powered Job Application Automator (Summarized)

## Overview

This project provides a **developer-first, human-in-the-loop web platform** for streamlined, AI-assisted job applications—designed for maximum flexibility, pluggability, and user control at every stage. The app enables developers to:

- Parse job postings (LinkedIn or external ATS)
- Generate tailored, concise cover letters using AI
- Autofill required application forms
- Upload resumes and related documents
- Maintain control at every step: review, pause, resume, or stop before any data is submitted

All automation defaults to user verification, with manual intervention required for CAPTCHA, 2FA, or anti-bot triggers.

---

## Architecture & Workflow

### 1️⃣ Backend: GraphQL API (Node.js + PostgreSQL)
- **Entities**: `User`, `JobApplication`, `JobSpec`, `SessionStep`
- **Features**:
    - Register/login with email (demo mode)
    - Start new application from a job URL
    - Extract and preview job spec (stubbed for now)
    - Workflow engine: Each step (e.g., field mapping, review, submission) is atomic and requires user approval
    - Audit log for every user/AI action
- **Tech**: Node.js, TypeScript, Apollo Server, Prisma ORM, PostgreSQL, Jest E2E tests

### 2️⃣ ATS Connector & Plugin System
- **ATS Connectors**: Modular adapters (e.g., LinkedIn, Greenhouse) to extract job details and drive custom workflows
- **Pluggability**: New connectors and AI modules can be added with zero impact on existing code
- **Plugin Registry**: In-memory (stage 1), moving toward dynamic, user-scoped, and sandboxed plugins in later stages

### 3️⃣ UI: Human-in-the-Loop Stepper (React)
- **Stepper UI**: Sequential steps for resume upload, job entry, AI review, user approval, and completion
- **Controls**: Next, Back, Pause, Stop—user in control at all times
- **Manual Fallback**: On CAPTCHA/2FA or unexpected form fields, the workflow pauses and prompts user for input
- **State Persistence**: All workflow state is resumable; user can pick up from last checkpoint

### 4️⃣ AI-Powered Cover Letter Generation
- **Persona**: Expert Cover Letter Assistant (see [Cover Letter Generator](#cover-letter-generator))
- **Workflow**:
    - Gathers key job and profile details via targeted prompts
    - Generates three versions of a tailored, concise cover letter (≤150 words, see below)
    - Emphasizes core strengths, measurable outcomes, and cultural fit in a lean, professional style
    - Uses **bold** to highlight key qualifications and results

---

## Cover Letter Generator

The AI Cover Letter module is designed to ensure your application stands out with clarity, brevity, and direct alignment to each role’s requirements.

**Key Features:**
- Information gathering: Company name, role, job spec, relevant docs
- Ambiguity detection: Requests clarification as needed
- Output: Three versions (150, 125, 100 words), each following strict format and tone
- Customisation: Emphasizes metrics, impact, and **bold** key skills

**Paragraph Structure (all versions):**
1. **Introduction**: Reference role & company, express genuine interest
2. **Fitness for the Role**: 2–3 core strengths, collaboration/leadership qualities, cultural fit
3. **What I Bring**: 2–3 relevant projects/achievements with measurable results
4. **Closing**: Reiterate enthusiasm, forward-looking statement
5. **Signature**: Identical in all versions

**Example Output:**  
See [examples below](#cover-letter-examples).

---

## Best Practices Checklist

- All automation is **user-approved, not auto-submitted**.
- Each step can be paused, stopped, or resumed.
- CAPTCHAs/2FA/manual steps always default to human intervention.
- Codebase is fully typed and E2E-tested.
- Every feature is designed for extensibility: plug in new ATSes, AI modules, or workflow steps without rewriting.
- Cover letter generator adheres to lean, metrics-first, evidence-based communication.

---

## Output Structure

- Backend: `/backend` – GraphQL API (Node.js, Apollo, Prisma, Jest)
- ATS Plugins: `/plugins` – Connector and registry implementations
- Frontend: `/frontend` – React Stepper UI (Cypress E2E)
- Cover Letter AI: `/ai` – Structured prompts, formatting, and output generator
- Tests: `/__tests__` and `/cypress/e2e`
- Docs & setup: `README.md`, `.env.example`

---

## Cover Letter Examples

### ✨ Version A – 149 words

**Dear [Company Name] Leadership Team,**

I’m writing to express my interest in the **[Role]** role at **[COMPANY]**. I thrive in environments that value **speed**, **clarity**, and **engineering craft**—qualities that appear deeply embedded in **[COMPANY]'s** culture.

At **[My Most Recent Relevant Company Name]**, I’ve led **platform-wide initiatives** including our migration to a **microservices architecture** and an **AI-powered codegen pipeline** that removed over 80% of boilerplate and consistently ships **fully tested, E2E-ready solutions** developers can build on with confidence.

I mentor engineers by leading through example—pairing regularly, supporting growth, and helping teams align around **shared patterns** and **scalable systems**. My work has contributed to multiple team promotions and improved delivery velocity.

I recognise this is a pivotal moment for **[Company Name]'s**, and I’m excited to help scale your platform while supporting **developer experience**, **fast feedback**, and continued innovation in [Company Industry].

Best regards,  
Desmond O’Leary  
desoleary@gmail.com | (403) 971-3401

### ✨ Version B – 124 words

**Dear [Company Name] Leadership Team,**

I’m excited to apply for the **[Role]** role at **[Company Name]**. Your focus on **scale**, **developer autonomy**, and product-led engineering strongly aligns with how I lead and deliver software.

At **[My Most Recent Relevant Company Name]**, I helped drive our transition to **microservices** and created an **AI-driven codegen pipeline** that eliminated 80%+ of boilerplate—enabling teams to ship **E2E-tested foundations** faster and with greater confidence.

I mentor engineers through pairing, shared architecture, and delivery patterns—supporting multiple team promotions and consistent velocity improvements.

I’d love to contribute hands-on to **[Company Name]'s** platform and help accelerate delivery while evolving systems that scale with trust.

Best regards,  
Desmond O’Leary  
desoleary@gmail.com | (403) 971-3401

### ✨ Version C – 100 words

**Dear [Company Name] Leadership Team,**

I’m excited about the **[Role]** opportunity at **[Company Name]**. Your focus on **scalability**, **developer experience**, and **modern architecture** reflects how I think about building great systems.

At **[My Most Recent Relevant Company Name]**, I led delivery of a **microservices migration** and an **AI-powered codegen pipeline** that cut 80% of boilerplate—shipping **E2E-tested systems** developers could build on immediately.

I mentor through pairing and shared patterns, helping teams scale, align, and deliver with confidence. I’d love to help **[Company Name]'s** move faster while supporting both near-term outcomes and long-term growth.

Best regards,  
Desmond O’Leary  
desoleary@gmail.com | (403) 971-3401

---

## Getting Started

1. **Clone the repo** and install dependencies for `/backend`, `/frontend`, and `/plugins`.
2. Set up PostgreSQL, create your `.env` files, and run migrations (`npx prisma migrate dev`).
3. Run backend and frontend in parallel.
4. Open the UI, upload your resume, paste a job URL, and follow the stepper to generate and review your application package.
5. Review, edit, or approve each step before any data is submitted.

---

## Contributing

- All modules are designed for safe, pluggable extension—PRs for new connectors, AI modules, or UI improvements are welcome.
- Please run the full E2E test suite before submitting changes.
- See `/CONTRIBUTING.md` for architectural guidelines and style conventions.

---

## License

MIT

---


