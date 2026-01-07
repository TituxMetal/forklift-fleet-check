# ForkliftFleetCheck — MVP Definition

## Overview

A forklift fleet management and daily inspection application. Operators perform pre-shift
inspections, supervisors monitor fleet health, and administrators manage equipment and inspection
checklists.

**Primary goal:** Practice and solidify clean/hexagonal architecture patterns with a real-world
business application.

**Secondary goal:** Implement a multi-role system (Operator, Supervisor, Admin) with role-based
access.

**UI Language:** French

## Core Value

1. **Daily inspections** — Operators complete safety checklists before using equipment
2. **Fleet visibility** — Supervisors see fleet status at a glance, track issues
3. **Configurable checklists** — Admins can manage equipment and inspection questions
4. **Compliance** — Keep records of all inspections for safety audits

## Reference

Functional prototype (Spark): [fleetchariot](https://github.com/TituxMetal/fleetchariot)

This prototype serves as the visual and functional reference. The real project will be rebuilt from
scratch with proper architecture.

---

## MVP Scope

### MVP Core (Must Ship — ~8 weeks)

All features from the Spark prototype, rebuilt properly with clean architecture.

### MVP Full (Nice to Have)

Enhanced reporting, export features, more granular permissions.

---

## MVP Core Features

### 1. Authentication & Roles

- [ ] User registration/login (Better Auth)
- [ ] Three roles: `operator`, `supervisor`, `admin`
- [ ] Role-based route protection
- [ ] Role-based UI (different views per role)

### 2. Equipment Management (Admin)

- [ ] Create equipment with details:
  - Name (e.g., "Transpalette T100")
  - ID/Code (e.g., "TP-100")
  - CACES category (1, 3, 5)
  - Ownership type (Propriété, Location courte durée, Location longue durée)
  - Status (OK, À surveiller, Hors service)
- [ ] Edit equipment
- [ ] Remove/deactivate equipment
- [ ] List all equipment with filters

### 3. Inspection Questions Management (Admin)

- [ ] Create inspection questions with:
  - Question text (e.g., "État général de la machine")
  - CACES category (Communes, CACES 1, CACES 3, CACES 5)
  - Possible answers with severity levels:
    - OK (green) — No issue
    - Défaut (orange) — To monitor
    - Critique (red) — Out of service
- [ ] Edit questions and answers
- [ ] Delete questions
- [ ] Questions grouped by category

### 4. Daily Inspection (Operator)

- [ ] View list of equipment (grouped by CACES category)
- [ ] See equipment status (OK, À surveiller from last inspection)
- [ ] Start inspection for an equipment
- [ ] Step-by-step questionnaire:
  - Progress indicator (Question X sur Y)
  - Single choice per question
  - Optional comment field for non-OK answers
  - Navigate between questions
- [ ] Submit inspection
- [ ] View personal inspection history

### 5. Inspection Results

- [ ] Calculate overall status based on answers:
  - All OK → Equipment status = OK
  - Any Défaut → Equipment status = À surveiller
  - Any Critique → Equipment status = Hors service
- [ ] Store inspection record with:
  - Operator
  - Equipment
  - Date/time
  - All answers with comments
  - Calculated status

### 6. Supervisor Dashboard

- [ ] KPIs:
  - Inspections today
  - Défauts signalés
  - Problèmes critiques
- [ ] Fleet status overview (all equipment with current status)
- [ ] Last inspection date per equipment
- [ ] Inspection history with filters:
  - By status
  - By equipment
  - By operator
  - Search
- [ ] View inspection details

### 7. Operator View

- [ ] Equipment list for daily inspection
- [ ] Personal inspection history
- [ ] Cannot access admin or supervisor features

---

## MVP Full Features (v1.1)

All of MVP Core, plus:

1. **More granular permissions** — Beyond simple roles if needed

---

## Out of Scope (for MVP)

- Export CSV/PDF
- Notifications
- Scheduling/calendar for inspections
- Maintenance work orders
- Spare parts inventory
- Mobile native app (responsive web is enough)
- Offline mode
- Multi-site/multi-company
- Advanced analytics/charts

---

## Technical Stack

| Layer                       | Technology                                 |
| --------------------------- | ------------------------------------------ |
| Monorepo                    | Turborepo                                  |
| Runtime and Package Manager | Bun 1.3.x                                  |
| Backend                     | NestJS 11.x (Clean/Hexagonal Architecture) |
| Database                    | SQLite                                     |
| ORM                         | Prisma 7.x                                 |
| Auth                        | Better Auth 1.4.x (with roles)             |
| Frontend                    | Astro 5.x + React 19.x                     |
| Styling                     | TailwindCSS v4 (dark theme)                |
| Testing                     | Bun test                                   |
| Linting                     | ESLint 9.x + Prettier 3.x                  |
| Git Hooks                   | Husky 9.x + CommitLint 20.x                |
| Deployment                  | Docker, docker-compose                     |

Starting point: Clone from [sample-project](https://github.com/TituxMetal/sample-project)

### Architecture

- **Backend:** Clean/Hexagonal Architecture (domain, application, infrastructure)
- **Frontend:** Feature-based folder structure
- **Testing:** TDD approach

### Git Workflow

- **Branching:** `main` ← `develop` ← `feature/*`, `fix/*`, `hotfix/*`
- **Commits:** Conventional commits, atomic
- **PRs:** Required for merging

---

## Data Model (High-Level)

```text
User
├── id
├── email
├── password (hashed)
├── name
├── role (operator | supervisor | admin)
└── inspections[]

Equipment
├── id
├── name
├── code
├── cacesCategory (1 | 3 | 5)
├── ownershipType (property | short_rental | long_rental)
├── status (ok | warning | out_of_service)
├── isActive
└── inspections[]

QuestionCategory
├── id
├── name (Communes | CACES 1 | CACES 3 | CACES 5)
├── sortOrder
└── questions[]

Question
├── id
├── categoryId
├── text
├── sortOrder
└── answers[]

Answer
├── id
├── questionId
├── text
├── severity (ok | warning | critical)
├── sortOrder

Inspection
├── id
├── equipmentId
├── operatorId
├── performedAt
├── status (ok | warning | out_of_service)
└── responses[]

InspectionResponse
├── id
├── inspectionId
├── questionId
├── answerId
├── comment (optional)
```

---

## "Done" Criteria

### MVP Core (Required)

- [ ] Deployed to a public URL
- [ ] Three roles work (operator, supervisor, admin)
- [ ] Admin can manage equipment
- [ ] Admin can manage inspection questions/answers
- [ ] Operator can perform step-by-step inspection
- [ ] Inspection calculates equipment status
- [ ] Operator sees personal history
- [ ] Supervisor dashboard shows fleet status and KPIs
- [ ] Supervisor can view all inspections with filters
- [ ] Mobile-friendly (responsive)
- [ ] Core features tested
- [ ] README with setup instructions

### MVP Full (Stretch Goal)

- [ ] All MVP Core criteria met
- [ ] More granular permissions beyond simple roles

---

## Build Order (High-Level)

> **Note:** Bird's-eye view. Each item = full feature (backend + frontend + tests). Keep dev
> sessions focused on ONE feature.

### MVP Core

1. **Project setup and deploy** — Clone sample-project, rename, configure, deploy skeleton
2. **Auth with roles** — Better Auth config, three roles, role-based guards
3. **Equipment CRUD** — Admin can manage equipment
4. **Question categories** — CRUD for categories
5. **Questions and answers** — CRUD with severity levels
6. **Operator equipment list** — View equipment grouped by CACES
7. **Inspection flow** — Step-by-step questionnaire, submit, calculate status
8. **Operator history** — Personal inspection list
9. **Supervisor dashboard** — KPIs, fleet overview
10. **Supervisor inspection history** — List with filters, detail view
11. **Polish** — UI refinements, responsive, final fixes

### MVP Full (if time permits)

All of MVP Core, plus:

1. **Granular permissions** — More fine-grained access control beyond roles
