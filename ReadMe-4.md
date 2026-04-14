# FlowBoard — Project Management SaaS

> **This document is the complete implementation specification.** Read it fully before writing any code. Follow every section in order. Do not skip steps, do not stub features, and do not move to the next phase until all tests for the current phase pass.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Project Structure](#project-structure)
4. [Implementation Phases](#implementation-phases)
   - [Phase 0: Scaffolding & Configuration](#phase-0-scaffolding--configuration)
   - [Phase 1: Database Schema & ORM](#phase-1-database-schema--orm)
   - [Phase 2: Authentication System](#phase-2-authentication-system)
   - [Phase 3: Organizations & Membership](#phase-3-organizations--membership)
   - [Phase 4: Projects](#phase-4-projects)
   - [Phase 5: Task Management](#phase-5-task-management)
   - [Phase 6: Comments & Activity Log](#phase-6-comments--activity-log)
   - [Phase 7: Real-Time Updates](#phase-7-real-time-updates)
   - [Phase 8: Notifications](#phase-8-notifications)
   - [Phase 9: Dashboard & Analytics](#phase-9-dashboard--analytics)
   - [Phase 10: Global Search](#phase-10-global-search)
   - [Phase 11: Seed Script](#phase-11-seed-script)
   - [Phase 12: Final QA & Hardening](#phase-12-final-qa--hardening)
5. [API Design Standards](#api-design-standards)
6. [Testing Strategy](#testing-strategy)
7. [Architecture Decisions](#architecture-decisions)

---

## Project Overview

FlowBoard is a fully functional, production-ready project management SaaS application. It supports multi-tenant organizations, role-based access control, real-time collaboration, and a rich task management interface with Kanban, list, and calendar views.

This is not a prototype. Every feature must be complete with validation, error handling, tests, and polished UI.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript (strict mode) |
| Database | SQLite via Prisma ORM |
| Styling | Tailwind CSS |
| Auth | Custom implementation using `bcryptjs` + JWT stored in HTTP-only cookies |
| Validation | Zod (all API inputs) |
| Real-time | Server-Sent Events (SSE) |
| Testing | Vitest (unit + integration), Playwright (e2e) |
| Linting | ESLint + Prettier |
| Drag & Drop | `@hello-pangea/dnd` (maintained fork of react-beautiful-dnd) |
| Charts | Recharts |
| Date handling | date-fns |
| Icons | lucide-react |

---

## Project Structure

```
flowboard/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   ├── signup/page.tsx
│   │   │   └── forgot-password/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx                  # Authenticated shell with sidebar
│   │   │   ├── page.tsx                    # User dashboard
│   │   │   ├── notifications/page.tsx
│   │   │   ├── search/page.tsx
│   │   │   └── org/
│   │   │       ├── [orgId]/
│   │   │       │   ├── settings/page.tsx
│   │   │       │   ├── members/page.tsx
│   │   │       │   └── projects/
│   │   │       │       ├── page.tsx        # Project list
│   │   │       │       └── [projectId]/
│   │   │       │           ├── page.tsx    # Kanban (default view)
│   │   │       │           ├── list/page.tsx
│   │   │       │           ├── calendar/page.tsx
│   │   │       │           └── settings/page.tsx
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   │   ├── signup/route.ts
│   │   │   │   ├── login/route.ts
│   │   │   │   ├── logout/route.ts
│   │   │   │   ├── me/route.ts
│   │   │   │   ├── forgot-password/route.ts
│   │   │   │   └── reset-password/route.ts
│   │   │   ├── organizations/
│   │   │   │   ├── route.ts                # GET (list), POST (create)
│   │   │   │   └── [orgId]/
│   │   │   │       ├── route.ts            # GET, PATCH, DELETE
│   │   │   │       ├── members/route.ts    # GET, POST (invite), PATCH (role), DELETE (remove)
│   │   │   │       └── projects/
│   │   │   │           ├── route.ts        # GET, POST
│   │   │   │           └── [projectId]/
│   │   │   │               ├── route.ts    # GET, PATCH, DELETE
│   │   │   │               ├── statuses/route.ts
│   │   │   │               └── tasks/
│   │   │   │                   ├── route.ts        # GET (with filters), POST
│   │   │   │                   └── [taskId]/
│   │   │   │                       ├── route.ts    # GET, PATCH, DELETE
│   │   │   │                       ├── comments/route.ts
│   │   │   │                       ├── subtasks/route.ts
│   │   │   │                       └── dependencies/route.ts
│   │   │   ├── notifications/route.ts
│   │   │   ├── search/route.ts
│   │   │   ├── activity/route.ts
│   │   │   ├── debug/
│   │   │   │   └── dump/route.ts          # Integration-test user dump (Phase 11)
│   │   │   └── sse/route.ts
│   │   └── layout.tsx
│   ├── components/
│   │   ├── ui/                             # Reusable primitives (Button, Input, Modal, etc.)
│   │   ├── auth/                           # Login/signup forms
│   │   ├── layout/                         # Sidebar, Header, NotificationBell
│   │   ├── projects/                       # ProjectCard, ProjectList
│   │   ├── tasks/                          # KanbanBoard, ListView, CalendarView, TaskCard, TaskModal
│   │   ├── comments/                       # CommentThread, CommentForm
│   │   ├── dashboard/                      # StatCards, ActivityFeed, Charts
│   │   └── search/                         # SearchBar, SearchResults
│   ├── lib/
│   │   ├── prisma.ts                       # Prisma client singleton
│   │   ├── auth.ts                         # JWT helpers, password hashing, session middleware
│   │   ├── validations/                    # Zod schemas per domain
│   │   │   ├── auth.ts
│   │   │   ├── organization.ts
│   │   │   ├── project.ts
│   │   │   ├── task.ts
│   │   │   └── comment.ts
│   │   ├── errors.ts                       # AppError class, error response helper
│   │   ├── rate-limit.ts                   # In-memory rate limiter
│   │   ├── sse.ts                          # SSE connection manager
│   │   ├── activity.ts                     # Activity log helper
│   │   └── permissions.ts                  # RBAC checks
│   ├── hooks/
│   │   ├── useSSE.ts
│   │   ├── useNotifications.ts
│   │   └── useDebounce.ts
│   └── types/
│       └── index.ts                        # Shared TypeScript types
├── tests/
│   ├── unit/
│   │   ├── lib/
│   │   │   ├── auth.test.ts
│   │   │   ├── permissions.test.ts
│   │   │   ├── rate-limit.test.ts
│   │   │   ├── validations.test.ts
│   │   │   └── activity.test.ts
│   │   └── components/
│   │       ├── TaskCard.test.tsx
│   │       └── KanbanBoard.test.tsx
│   ├── integration/
│   │   ├── api/
│   │   │   ├── auth.test.ts
│   │   │   ├── organizations.test.ts
│   │   │   ├── projects.test.ts
│   │   │   ├── tasks.test.ts
│   │   │   ├── comments.test.ts
│   │   │   ├── notifications.test.ts
│   │   │   └── search.test.ts
│   │   └── helpers.ts                      # Test DB setup, auth helpers
│   └── e2e/
│       ├── auth.spec.ts
│       ├── project-management.spec.ts
│       ├── task-workflow.spec.ts
│       ├── kanban-dnd.spec.ts
│       ├── search.spec.ts
│       └── helpers/
│           └── fixtures.ts
├── .eslintrc.json
├── .prettierrc
├── playwright.config.ts
├── vitest.config.ts
├── tsconfig.json
├── tailwind.config.ts
├── next.config.js
├── package.json
└── README.md
```

---

## Implementation Phases

> **CRITICAL RULE:** After completing each phase, run the full test suite (`npm run test && npm run lint && npm run typecheck`). Fix every failure before moving to the next phase. If a fix in one phase breaks a test from a prior phase, fix that regression immediately.

---

### Phase 0: Scaffolding & Configuration

**Goal:** Set up the project skeleton so everything compiles and runs with zero errors before any features are built.

**Steps:**

1. Initialize the project:
   ```bash
   npx create-next-app@14 flowboard --typescript --tailwind --eslint --app --src-dir --no-import-alias
   cd flowboard
   ```

2. Install all dependencies at once:
   ```bash
   npm install prisma @prisma/client bcryptjs jsonwebtoken zod date-fns recharts lucide-react @hello-pangea/dnd
   npm install -D @types/bcryptjs @types/jsonwebtoken vitest @vitejs/plugin-react @testing-library/react @testing-library/jest-dom jsdom playwright @playwright/test prettier eslint-config-prettier
   ```

3. Configure TypeScript `tsconfig.json` — enable `strict: true`, set path aliases `@/*` → `src/*`.

4. Configure ESLint (`.eslintrc.json`): extend `next/core-web-vitals` and `prettier`. Add rules: `no-unused-vars: error`, `no-console: warn`.

5. Configure Prettier (`.prettierrc`): `semi: true`, `singleQuote: true`, `trailingComma: "es5"`, `printWidth: 100`.

6. Configure Vitest (`vitest.config.ts`): environment `jsdom`, path aliases matching tsconfig, coverage thresholds at 80%.

7. Configure Playwright (`playwright.config.ts`): base URL `http://localhost:3000`, `webServer` block to auto-start the dev server, retries 2, screenshot on failure.

8. Add scripts to `package.json`:
   ```json
   {
     "dev": "next dev",
     "build": "next build",
     "start": "next start",
     "lint": "eslint src/ --ext .ts,.tsx --max-warnings 0",
     "format": "prettier --write 'src/**/*.{ts,tsx}'",
     "typecheck": "tsc --noEmit",
     "test": "vitest run",
     "test:watch": "vitest",
     "test:e2e": "playwright test",
     "test:all": "npm run typecheck && npm run lint && npm run test && npm run test:e2e",
     "db:push": "prisma db push",
     "db:seed": "tsx prisma/seed.ts",
     "db:studio": "prisma studio"
   }
   ```

9. Create placeholder `src/lib/prisma.ts` with singleton pattern.

10. Verify: `npm run typecheck && npm run lint` must pass with zero errors.

---

### Phase 1: Database Schema & ORM

**Goal:** Define the complete data model up front. Every table, relation, index, and enum.

**Prisma schema (`prisma/schema.prisma`):**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model User {
  id                String    @id @default(cuid())
  email             String    @unique
  name              String
  passwordHash      String
  avatarUrl         String?
  emailVerified     Boolean   @default(false)
  resetToken        String?
  resetTokenExpiry  DateTime?
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt

  memberships       Membership[]
  assignedTasks     Task[]          @relation("TaskAssignee")
  createdTasks      Task[]          @relation("TaskCreator")
  comments          Comment[]
  activities        Activity[]
  notifications     Notification[]
}

model Organization {
  id          String   @id @default(cuid())
  name        String
  slug        String   @unique
  description String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  members     Membership[]
  projects    Project[]
}

model Membership {
  id        String   @id @default(cuid())
  role      String   @default("member")  // owner, admin, member, viewer
  createdAt DateTime @default(now())

  userId    String
  orgId     String
  user      User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  org       Organization @relation(fields: [orgId], references: [id], onDelete: Cascade)

  @@unique([userId, orgId])
  @@index([orgId])
}

model Project {
  id          String   @id @default(cuid())
  name        String
  description String?
  archived    Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  orgId       String
  org         Organization @relation(fields: [orgId], references: [id], onDelete: Cascade)

  statuses    Status[]
  tasks       Task[]
  labels      Label[]

  @@index([orgId])
}

model Status {
  id        String @id @default(cuid())
  name      String
  color     String @default("#6B7280")
  position  Int
  projectId String
  project   Project @relation(fields: [projectId], references: [id], onDelete: Cascade)

  tasks     Task[]

  @@index([projectId])
}

model Label {
  id        String @id @default(cuid())
  name      String
  color     String @default("#3B82F6")
  projectId String
  project   Project @relation(fields: [projectId], references: [id], onDelete: Cascade)

  tasks     TaskLabel[]

  @@index([projectId])
}

model Task {
  id          String    @id @default(cuid())
  title       String
  description String?
  priority    String    @default("medium")  // urgent, high, medium, low, none
  dueDate     DateTime?
  position    Int
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  statusId    String
  status      Status    @relation(fields: [statusId], references: [id])
  projectId   String
  project     Project   @relation(fields: [projectId], references: [id], onDelete: Cascade)
  assigneeId  String?
  assignee    User?     @relation("TaskAssignee", fields: [assigneeId], references: [id])
  creatorId   String
  creator     User      @relation("TaskCreator", fields: [creatorId], references: [id])
  parentId    String?
  parent      Task?     @relation("Subtasks", fields: [parentId], references: [id])

  subtasks    Task[]    @relation("Subtasks")
  labels      TaskLabel[]
  comments    Comment[]
  activities  Activity[]

  // Dependencies
  blockedBy   TaskDependency[] @relation("BlockedTask")
  blocking    TaskDependency[] @relation("BlockingTask")

  @@index([projectId])
  @@index([statusId])
  @@index([assigneeId])
  @@index([parentId])
}

model TaskLabel {
  taskId  String
  labelId String
  task    Task  @relation(fields: [taskId], references: [id], onDelete: Cascade)
  label   Label @relation(fields: [labelId], references: [id], onDelete: Cascade)

  @@id([taskId, labelId])
}

model TaskDependency {
  id             String @id @default(cuid())
  blockedTaskId  String
  blockingTaskId String
  blockedTask    Task   @relation("BlockedTask", fields: [blockedTaskId], references: [id], onDelete: Cascade)
  blockingTask   Task   @relation("BlockingTask", fields: [blockingTaskId], references: [id], onDelete: Cascade)

  @@unique([blockedTaskId, blockingTaskId])
}

model Comment {
  id        String   @id @default(cuid())
  content   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  taskId    String
  task      Task     @relation(fields: [taskId], references: [id], onDelete: Cascade)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  parentId  String?
  parent    Comment? @relation("Replies", fields: [parentId], references: [id])

  replies   Comment[] @relation("Replies")

  @@index([taskId])
  @@index([parentId])
}

model Activity {
  id          String   @id @default(cuid())
  action      String   // created, updated, deleted, commented, status_changed, assigned, etc.
  entityType  String   // task, project, comment, membership
  entityId    String
  metadata    String?  // JSON string with before/after values
  createdAt   DateTime @default(now())

  userId      String
  user        User     @relation(fields: [userId], references: [id])
  taskId      String?
  task        Task?    @relation(fields: [taskId], references: [id], onDelete: SetNull)

  @@index([entityType, entityId])
  @@index([userId])
  @@index([taskId])
  @@index([createdAt])
}

model Notification {
  id        String   @id @default(cuid())
  type      String   // assigned, commented, mentioned, status_changed, invited
  message   String
  read      Boolean  @default(false)
  link      String?
  metadata  String?  // JSON
  createdAt DateTime @default(now())

  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId, read])
  @@index([createdAt])
}
```

**Steps:**

1. Create `.env` file: `DATABASE_URL="file:./dev.db"`
2. Write the schema above.
3. Run `npx prisma db push` — must succeed.
4. Run `npx prisma generate` — must succeed.
5. Implement `src/lib/prisma.ts` singleton.
6. Write unit tests that verify the Prisma client can connect and perform basic CRUD on every model.
7. Run tests — all must pass.

---

### Phase 2: Authentication System

**Goal:** Complete auth flow — signup, login, logout, session management, password reset.

**Implementation details:**

1. **Password hashing:** `bcryptjs` with 12 salt rounds.

2. **JWT tokens:** Signed with `process.env.JWT_SECRET`, 7-day expiry. Payload: `{ userId: string, email: string }`. Stored in HTTP-only, secure, sameSite=lax cookie named `flowboard-token`.

3. **Auth middleware** (`src/lib/auth.ts`):
   - `hashPassword(password: string): Promise<string>`
   - `verifyPassword(password: string, hash: string): Promise<boolean>`
   - `generateToken(user: { id: string, email: string }): string`
   - `verifyToken(token: string): JWTPayload | null`
   - `getCurrentUser(request: Request): Promise<User | null>` — reads cookie, verifies JWT, fetches user from DB.
   - `requireAuth(request: Request): Promise<User>` — throws 401 if not authenticated.

4. **Zod schemas** (`src/lib/validations/auth.ts`):
   - `signupSchema`: email (valid email, lowercase trimmed), name (2-100 chars), password (min 8, must include uppercase, lowercase, number).
   - `loginSchema`: email, password.
   - `forgotPasswordSchema`: email.
   - `resetPasswordSchema`: token, newPassword.

5. **API routes:**
   - `POST /api/auth/signup` — validate, check email uniqueness, hash password, create user, generate token, set cookie, return user (without hash).
   - `POST /api/auth/login` — validate, find user by email, verify password, generate token, set cookie.
   - `POST /api/auth/logout` — clear cookie.
   - `GET /api/auth/me` — return current user or 401.
   - `POST /api/auth/forgot-password` — generate reset token (random 64-char hex), store hashed version in DB with 1-hour expiry. Return success message regardless (prevent email enumeration). Log the token to console (simulating email).
   - `POST /api/auth/reset-password` — validate token, check expiry, update password, clear token.

6. **Pages:**
   - `/login` — email + password form, link to signup and forgot password.
   - `/signup` — name + email + password form, client-side validation matching Zod schemas.
   - `/forgot-password` — email input, success message.
   - Auth pages redirect to dashboard if already logged in.

7. **Tests:**
   - Unit: all auth utility functions.
   - Integration: every API route (success + every error case: invalid input, duplicate email, wrong password, expired token, etc.).
   - E2E: full signup → login → logout → password reset flow.

8. Run full test suite — fix all failures.

---

### Phase 3: Organizations & Membership

**Goal:** Multi-tenant org system with invite-based membership and RBAC.

**Roles and permissions:**

| Permission | Owner | Admin | Member | Viewer |
|---|---|---|---|---|
| Delete org | ✅ | ❌ | ❌ | ❌ |
| Edit org settings | ✅ | ✅ | ❌ | ❌ |
| Manage members (invite, remove, change role) | ✅ | ✅ | ❌ | ❌ |
| Create/archive projects | ✅ | ✅ | ✅ | ❌ |
| Create/edit/delete tasks | ✅ | ✅ | ✅ | ❌ |
| Comment on tasks | ✅ | ✅ | ✅ | ❌ |
| View everything | ✅ | ✅ | ✅ | ✅ |

**Implementation details:**

1. **Permissions helper** (`src/lib/permissions.ts`):
   - `getUserMembership(userId, orgId): Promise<Membership | null>`
   - `requireRole(userId, orgId, minimumRole): Promise<Membership>` — throws 403 if insufficient.
   - Role hierarchy: owner > admin > member > viewer.

2. **Slug generation:** From org name, lowercased, spaces → hyphens, strip non-alphanumeric, append random 4-char suffix if collision.

3. **API routes:**
   - `GET /api/organizations` — list orgs where user is a member.
   - `POST /api/organizations` — create org, creator becomes owner, create 4 default statuses (Backlog, To Do, In Progress, Done).
   - `GET /api/organizations/[orgId]` — org details (requires member).
   - `PATCH /api/organizations/[orgId]` — update name/description (requires admin).
   - `DELETE /api/organizations/[orgId]` — delete org (requires owner). Cascades.
   - `GET /api/organizations/[orgId]/members` — list members with roles.
   - `POST /api/organizations/[orgId]/members` — invite by email. If user exists, add membership. If not, return error prompting them to sign up first. (requires admin).
   - `PATCH /api/organizations/[orgId]/members` — change member role (requires admin, cannot change owner).
   - `DELETE /api/organizations/[orgId]/members` — remove member (requires admin, cannot remove owner).

4. **Pages:**
   - Org creation modal/page.
   - Org settings page (name, description, danger zone with delete).
   - Members page with role badges, invite form, role change dropdown, remove button.

5. **Tests:**
   - Unit: all permission functions.
   - Integration: every API route with every role (verify allowed + denied cases).
   - E2E: create org → invite member → change role → remove member.

6. Run full test suite — fix all failures.

---

### Phase 4: Projects

**Goal:** Project CRUD with custom statuses and archive capability.

**Implementation details:**

1. **Default statuses** created when a project is created:
   - Backlog (position 0, color #6B7280)
   - To Do (position 1, color #3B82F6)
   - In Progress (position 2, color #F59E0B)
   - Done (position 3, color #10B981)

2. **API routes:**
   - `GET /api/organizations/[orgId]/projects` — list non-archived projects. Query param `?archived=true` to list archived ones.
   - `POST /api/organizations/[orgId]/projects` — create project + default statuses (requires member).
   - `GET /api/organizations/[orgId]/projects/[projectId]` — project details with statuses and task counts per status.
   - `PATCH /api/organizations/[orgId]/projects/[projectId]` — update name, description, archived (requires member).
   - `DELETE /api/organizations/[orgId]/projects/[projectId]` — delete project and all tasks (requires admin).
   - `GET /api/.../projects/[projectId]/statuses` — list statuses ordered by position.
   - `PATCH /api/.../projects/[projectId]/statuses` — bulk update statuses (reorder, rename, add, remove). Cannot remove a status that has tasks unless a migration target is specified.

3. **Pages:**
   - Project list page with cards showing name, description, task count, last updated.
   - Project settings page: edit name/description, manage statuses (reorder via drag, rename, change color, add/remove), archive toggle, danger zone delete.

4. **Tests:**
   - Integration: all project and status API routes.
   - E2E: create project → customize statuses → archive → unarchive.

5. Run full test suite — fix all failures.

---

### Phase 5: Task Management

**Goal:** Full task CRUD, subtasks, dependencies, labels, and three view modes (Kanban, list, calendar).

**This is the largest phase. Take it methodically.**

**Implementation details:**

1. **Task fields:** title (required, 1-500 chars), description (optional, markdown-compatible plain text), priority (urgent/high/medium/low/none), dueDate (optional ISO date), assigneeId (optional, must be org member), statusId (required, must belong to project), position (integer for ordering within a status column), labels (many-to-many via TaskLabel).

2. **Task API routes:**
   - `GET /api/.../tasks` — list tasks for project. Support query params:
     - `status` — filter by status ID
     - `assignee` — filter by assignee ID
     - `priority` — filter by priority
     - `label` — filter by label ID
     - `search` — search title/description
     - `dueBefore` / `dueAfter` — date range
     - `sort` — `position` (default), `dueDate`, `priority`, `createdAt`
     - `parentId` — filter subtasks of a specific task (use `parentId=null` for top-level only)
   - `POST /api/.../tasks` — create task (requires member). Auto-set position to max+1 in target status. Log activity.
   - `GET /api/.../tasks/[taskId]` — full task details with assignee, creator, labels, subtasks, dependencies, comment count.
   - `PATCH /api/.../tasks/[taskId]` — update any field. If statusId changes, log `status_changed` activity with before/after. If assigneeId changes, log `assigned` activity and create notification.
   - `DELETE /api/.../tasks/[taskId]` — delete task and subtasks (requires member). Log activity.

3. **Subtasks API:**
   - `GET /api/.../tasks/[taskId]/subtasks` — list subtasks.
   - `POST /api/.../tasks/[taskId]/subtasks` — create subtask (parentId set to taskId). Subtasks inherit project and get a default status.

4. **Dependencies API:**
   - `GET /api/.../tasks/[taskId]/dependencies` — list tasks this task is blocked by, and tasks this task is blocking.
   - `POST /api/.../tasks/[taskId]/dependencies` — add dependency. Body: `{ blockingTaskId }`. Validate: no self-reference, no circular dependencies (implement cycle detection), both tasks in same project.
   - `DELETE /api/.../tasks/[taskId]/dependencies/[depId]` — remove dependency.

5. **Labels:** Managed at project level.
   - CRUD via project settings or inline task editing.

6. **Kanban Board view** (`/org/[orgId]/projects/[projectId]`):
   - Columns = statuses ordered by position.
   - Cards show: title, priority indicator (colored dot/badge), assignee avatar, due date (red if overdue), subtask count, comment count, label chips.
   - Drag-and-drop between columns (changes status + position) and within columns (reorder). Use `@hello-pangea/dnd`. On drop, update task via API (PATCH status + position).
   - "Add task" button at bottom of each column — inline quick-add with just title, then open modal for full editing.

7. **List view** (`/org/[orgId]/projects/[projectId]/list`):
   - Table with columns: checkbox (for future bulk actions), title, status (dropdown), priority (dropdown), assignee (dropdown), due date (date picker), labels.
   - Sortable columns. Filterable by any column.

8. **Calendar view** (`/org/[orgId]/projects/[projectId]/calendar`):
   - Monthly calendar grid. Tasks placed on their due date. Click to open task modal. Tasks without due dates shown in a sidebar list.

9. **Task modal/detail view:**
   - Opens as a modal overlay or slide-over panel.
   - Full edit capability: title, description (textarea), status dropdown, priority dropdown, assignee dropdown (org members), due date picker, labels multi-select.
   - Subtasks section: list with add form, click to expand.
   - Dependencies section: "Blocked by" and "Blocking" lists with add/remove.
   - Comments section (implemented in Phase 6).
   - Activity log sidebar (implemented in Phase 6).

10. **Tests:**
    - Unit: cycle detection for dependencies, position calculation helpers.
    - Integration: all task, subtask, and dependency API routes. Test every filter combination. Test permission enforcement.
    - E2E: create task → edit → move on Kanban (drag-and-drop) → add subtask → add dependency → switch views.

11. Run full test suite — fix all failures.

---

### Phase 6: Comments & Activity Log

**Goal:** Threaded comments on tasks with @mentions, and a comprehensive activity/audit log.

**Implementation details:**

1. **Comments API:**
   - `GET /api/.../tasks/[taskId]/comments` — list comments for task, including nested replies (2 levels deep). Include author name and avatar.
   - `POST /api/.../tasks/[taskId]/comments` — create comment. Body: `{ content, parentId? }`. Detect @mentions using regex `@(\w+)` — find users by name in the org, create notifications for them. Log activity.
   - `PATCH /api/.../tasks/[taskId]/comments/[commentId]` — edit own comment only. Store `updatedAt`.
   - `DELETE /api/.../tasks/[taskId]/comments/[commentId]` — delete own comment (or admin can delete any).

2. **Comment UI:**
   - Threaded display: top-level comments with replies indented.
   - Comment form with textarea. Show "edited" indicator for edited comments.
   - Relative timestamps ("2 minutes ago", "yesterday").

3. **@Mentions:**
   - In comment textarea, typing `@` triggers a dropdown of org members (filter as you type).
   - Mentions rendered as styled badges/links in the comment display.
   - Mentioned users receive a notification.

4. **Activity log** (`src/lib/activity.ts`):
   - `logActivity({ userId, action, entityType, entityId, taskId?, metadata? })` — creates Activity record.
   - Metadata stores JSON with context: `{ before: { status: "To Do" }, after: { status: "In Progress" } }`.
   - Logged events: task created, task updated (each field change), task deleted, status changed, assignee changed, comment added, comment deleted, member invited, member removed, project created, project archived.

5. **Activity UI:**
   - In task modal: timeline of activity for that task.
   - In dashboard: recent activity across all user's projects.
   - Human-readable messages: "Alice changed status from To Do to In Progress", "Bob assigned this task to Charlie".

6. **Activity API:**
   - `GET /api/activity?entityType=task&entityId=xxx` — activity for a specific entity.
   - `GET /api/activity?orgId=xxx&limit=50` — recent activity for an org.

7. **Tests:**
   - Integration: comment CRUD, mention detection, activity logging.
   - E2E: add comment → reply → @mention → verify notification → edit → delete.

8. Run full test suite — fix all failures.

---

### Phase 7: Real-Time Updates

**Goal:** Use Server-Sent Events so multiple users see changes live without polling.

**Implementation details:**

1. **SSE Manager** (`src/lib/sse.ts`):
   - Maintains a `Map<string, Set<ReadableStreamController>>` keyed by org ID.
   - `addClient(orgId, controller)` — registers a new client.
   - `removeClient(orgId, controller)` — cleans up on disconnect.
   - `broadcast(orgId, event)` — sends event to all connected clients for that org.
   - Events are JSON: `{ type: "task_updated" | "task_created" | "task_deleted" | "comment_added" | "notification", payload: {...} }`.

2. **SSE API route** (`GET /api/sse`):
   - Requires auth. Query param: `orgId`.
   - Returns `ReadableStream` with `text/event-stream` content type.
   - Sends heartbeat ping every 30 seconds to keep connection alive.
   - On client disconnect, clean up.

3. **Client hook** (`src/hooks/useSSE.ts`):
   - Connects to `/api/sse?orgId=xxx`.
   - Parses events and dispatches to appropriate handlers.
   - Auto-reconnect with exponential backoff on disconnect.
   - Returns `{ lastEvent, isConnected }`.

4. **Integration points:** Every API route that mutates data (create/update/delete tasks, comments, status changes, etc.) broadcasts the change via SSE after the DB write succeeds.

5. **Client-side consumption:**
   - Kanban board listens for `task_updated`/`task_created`/`task_deleted` and updates the board in real-time without full refetch.
   - Comment section listens for `comment_added`.
   - Notification bell listens for `notification`.

6. **Tests:**
   - Unit: SSE manager (add/remove/broadcast).
   - Integration: connect to SSE endpoint, trigger a task update, verify event received.

7. Run full test suite — fix all failures.

---

### Phase 8: Notifications

**Goal:** In-app notification system with bell icon, dropdown, and read/unread management.

**Implementation details:**

1. **Notification triggers** (created via a helper `createNotification({ userId, type, message, link, metadata })`):
   - `assigned` — "Alice assigned you to 'Fix login bug'" — when a task is assigned to the user.
   - `commented` — "Bob commented on 'Fix login bug'" — when someone comments on a task the user is assigned to or has commented on.
   - `mentioned` — "Charlie mentioned you in a comment on 'Fix login bug'" — @mention in a comment.
   - `status_changed` — "The status of 'Fix login bug' changed to Done" — when a task the user is assigned to changes status.
   - `invited` — "You were added to org 'Acme Corp'" — org membership invite.
   - Do NOT notify the user about their own actions.

2. **API routes:**
   - `GET /api/notifications` — list notifications for current user, ordered by createdAt desc. Query params: `unreadOnly=true`, `limit=20`, `cursor` (for pagination).
   - `PATCH /api/notifications` — mark notifications as read. Body: `{ ids: string[] }` or `{ all: true }`.
   - `DELETE /api/notifications` — delete notifications. Body: `{ ids: string[] }`.

3. **UI components:**
   - **NotificationBell** in header: bell icon with unread count badge (red dot with number).
   - **Notification dropdown**: click bell to show dropdown with notification list. Each item shows: icon (by type), message, relative time, unread indicator (blue dot). Click to navigate to the linked entity and mark as read.
   - **Notifications page** (`/notifications`): full list with pagination, "Mark all read" button, filter by type.

4. **Real-time:** New notifications pushed via SSE. Bell count updates instantly.

5. **Tests:**
   - Integration: notification creation triggers, API routes, read/unread toggling.
   - E2E: assign task to another user → verify notification appears in bell → click → verify navigation.

6. Run full test suite — fix all failures.

---

### Phase 9: Dashboard & Analytics

**Goal:** Personal dashboard showing actionable information and project analytics.

**Implementation details:**

1. **Dashboard page** (`/` when authenticated):
   - **Stat cards row:** Total tasks assigned to me, Overdue tasks, Tasks due this week, Completed this week.
   - **My tasks table:** List of tasks assigned to current user across all orgs/projects, sorted by due date. Columns: task title (link), project name, status badge, priority, due date (red if overdue). Filterable by org.
   - **Recent activity feed:** Last 20 activity items across user's orgs. Same format as activity log.
   - **Charts section:**
     - Tasks by status (horizontal bar chart or donut chart) — per project, selectable via dropdown.
     - Tasks completed over time (line chart, last 30 days).
     - Tasks by priority (pie chart).

2. **Charts:** Use Recharts. Responsive containers, clean colors matching Tailwind palette, proper legends and tooltips.

3. **API support:** Dashboard data can be fetched via existing endpoints with appropriate filters, or create a dedicated `GET /api/dashboard` endpoint that aggregates:
   - Tasks assigned to current user (with overdue count).
   - Task counts by status for each project.
   - Completion trend (tasks moved to "Done" status per day for last 30 days).

4. **Tests:**
   - Integration: dashboard data endpoint returns correct aggregations.
   - E2E: after creating tasks with various states, verify dashboard shows correct counts and charts render.

5. Run full test suite — fix all failures.

---

### Phase 10: Global Search

**Goal:** Search across projects, tasks, and comments with filters.

**Implementation details:**

1. **Search API** (`GET /api/search`):
   - Query params: `q` (search term, required, min 2 chars), `type` (all/tasks/projects/comments), `orgId` (optional filter).
   - Search strategy (SQLite):
     - Tasks: search `title` and `description` using `LIKE %query%`.
     - Projects: search `name` and `description`.
     - Comments: search `content`.
   - Results include: entity type, title/snippet (highlight matched text), link, project name, org name, timestamp.
   - Paginated: `limit=20`, `cursor` for pagination.
   - Only return results the user has access to (member of the org).

2. **UI:**
   - **Search bar** in the header/sidebar: input with keyboard shortcut (Cmd/Ctrl+K).
   - **Search command palette** (modal overlay): opens on shortcut, type to search, results grouped by type (Projects, Tasks, Comments), navigate with arrow keys, Enter to go.
   - **Search results page** (`/search?q=xxx`): full results with type filter tabs, result cards with context snippets.

3. **Tests:**
   - Integration: search with various queries, verify access control (can't see results from orgs user isn't in), test filters.
   - E2E: use search bar → type query → click result → verify navigation.

4. Run full test suite — fix all failures.

---

### Phase 11: Seed Script

**Goal:** Populate the database with realistic demo data for testing and demo purposes.

**Seed script (`prisma/seed.ts`):**

Create the following data programmatically (use realistic names, descriptions, and dates):

1. **Users** (6):
   - Alice Johnson (alice@example.com) — will be owner of org 1
   - Bob Smith (bob@example.com) — admin
   - Charlie Davis (charlie@example.com) — member
   - Diana Wilson (diana@example.com) — member
   - Eve Martinez (eve@example.com) — viewer
   - Frank Brown (frank@example.com) — owner of org 2
   - All passwords: `Password123`

2. **Organizations** (2):
   - "Acme Corporation" (slug: acme-corp) — members: Alice (owner), Bob (admin), Charlie (member), Diana (member), Eve (viewer)
   - "Startup Labs" (slug: startup-labs) — members: Frank (owner), Alice (member), Bob (member)

3. **Projects** (5 total):
   - Acme Corp: "Website Redesign", "Mobile App v2", "Q4 Marketing Campaign"
   - Startup Labs: "MVP Development", "Customer Research"
   - Each project gets the 4 default statuses plus 1-2 custom statuses.
   - Each project gets 3-5 labels (e.g., "bug", "feature", "design", "urgent", "documentation").

4. **Tasks** (120+ total across all projects):
   - Distribute across statuses: ~25% Backlog, ~25% To Do, ~30% In Progress, ~20% Done.
   - Vary priorities: ~10% urgent, ~20% high, ~40% medium, ~20% low, ~10% none.
   - ~60% of tasks have assignees.
   - ~40% have due dates (some past = overdue, some this week, some next month).
   - ~30% of tasks have 1-3 subtasks each.
   - ~15% of tasks have dependencies.
   - Each task has 1-3 labels assigned.
   - Titles and descriptions should be realistic project management content (e.g., "Implement user authentication flow", "Design landing page hero section", "Fix mobile responsive issues on checkout").

5. **Comments** (200+ total):
   - Distribute across tasks: some have 0, most have 1-3, a few have 5-10.
   - Include threaded replies (some comments have 1-2 replies).
   - Some comments include @mentions.

6. **Activity log** (auto-generated from creating the above).

7. **Notifications** (30+):
   - Generate realistic notifications for various users.

8. **Push to Github for Final Check** Push this and all changes to Github so it can go through final checks and be saved.

**Run:** `npx tsx prisma/seed.ts` — must complete without errors. Verify by starting the app and browsing.

---

### Phase 12: Final QA & Hardening

**Goal:** Everything passes, everything is polished, nothing is stubbed.

**Steps:**

1. **Run the full quality pipeline:**
   ```bash
   npm run typecheck     # Zero TypeScript errors
   npm run lint          # Zero ESLint warnings or errors
   npm run format        # Prettier formats all files
   npm run test          # All Vitest tests pass
   npm run test:e2e      # All Playwright tests pass
   npm run build         # Next.js production build succeeds
   ```
   Fix every issue. Re-run until all six commands pass cleanly.

2. **Code audit — find and fix:**
   - Any `TODO`, `FIXME`, `HACK`, `XXX` comments → implement or remove.
   - Any `console.log` statements → remove or convert to proper error handling.
   - Any hardcoded strings that should be constants or env vars.
   - Any `any` types in TypeScript → replace with proper types.
   - Any API route missing input validation → add Zod validation.
   - Any API route missing error handling → add try/catch with proper error responses.
   - Any missing loading states in UI → add skeleton loaders or spinners.
   - Any missing empty states (e.g., no projects, no tasks) → add helpful empty state messages with CTAs.
   - Any missing error states in UI → add error boundaries or error messages.

3. **Security audit:**
   - Verify all API routes check authentication.
   - Verify all org-scoped routes check membership.
   - Verify all role-restricted actions check roles.
   - Verify no sensitive data (password hashes, tokens) leaks in API responses.
   - Verify rate limiting is applied to auth routes.

4. **UI polish:**
   - Consistent spacing, colors, typography across all pages.
   - All buttons have hover/active/disabled states.
   - All forms show validation errors inline.
   - All destructive actions have confirmation modals.
   - Responsive layout works on mobile viewport (min 375px).

5. **Final test run:**
   ```bash
   npm run test:all
   ```
   Must pass with zero failures.

6. **Seed and manual smoke test:**
   ```bash
   npx prisma db push --force-reset
   npm run db:seed
   npm run dev
   ```
   Navigate every page. Create a task, move it, comment, search. Verify everything works.

---

## API Design Standards

Every API route must follow these patterns:

```typescript
// Standard response format
type ApiResponse<T> = {
  data: T;
} | {
  error: {
    message: string;
    code: string;      // e.g., "VALIDATION_ERROR", "NOT_FOUND", "FORBIDDEN"
    details?: unknown;  // Zod errors, etc.
  };
};

// Standard error codes and HTTP statuses:
// 400 — VALIDATION_ERROR (invalid input)
// 401 — UNAUTHORIZED (not logged in)
// 403 — FORBIDDEN (insufficient permissions)
// 404 — NOT_FOUND
// 409 — CONFLICT (duplicate, etc.)
// 429 — RATE_LIMITED
// 500 — INTERNAL_ERROR
```

**Rate limiting** (`src/lib/rate-limit.ts`):
- In-memory token bucket.
- Auth routes: 10 requests per minute per IP.
- Other routes: 100 requests per minute per user.
- Return `429` with `Retry-After` header.

---

## Testing Strategy

| Test Type | Tool | Location | What to Test |
|---|---|---|---|
| Unit | Vitest | `tests/unit/` | Pure functions, utility helpers, validation schemas, permission logic, React components (isolated) |
| Integration | Vitest | `tests/integration/` | API routes with real DB (use test database), request → response validation, auth/permission enforcement |
| E2E | Playwright | `tests/e2e/` | Full user flows through the browser, multi-page interactions, drag-and-drop, real-time updates |

**Test database:** Integration tests use a separate SQLite file (`test.db`). Before each test file, reset the DB with `prisma db push --force-reset`. Use a helper to create test users and auth tokens.

**Coverage target:** 80% line coverage minimum across unit + integration tests.

---

## Architecture Decisions

1. **SQLite over Postgres:** Simpler setup, no external dependencies, sufficient for a single-instance SaaS demo. Prisma abstracts the SQL dialect, so migration to Postgres later is straightforward.

2. **Custom auth over NextAuth:** More control over the JWT structure, cookie handling, and password reset flow. Avoids the complexity of OAuth providers for this use case.

3. **SSE over WebSockets:** Simpler server-side implementation with Next.js App Router. Unidirectional (server → client) is sufficient since all mutations go through REST API routes. No need for a separate WebSocket server.

4. **In-memory rate limiting:** Acceptable for single-instance deployment. For multi-instance, replace with Redis-backed limiter.

5. **Monolithic Next.js:** API routes colocated with the frontend. Simpler deployment, shared types, no CORS configuration needed.

---

## Quick Start (After Implementation)

```bash
# Install dependencies
npm install

# Set up environment
cp .env.example .env

# Initialize database
npx prisma db push

# Seed demo data
npm run db:seed

# Start development server
npm run dev

# Open http://localhost:3000
# Login: alice@example.com / Password123
```