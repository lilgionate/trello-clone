# Trello‑Style Kanban

An opinionated, production‑ready Trello‑style Kanban board built with **Next.js 14 App Router**, **TypeScript**, **Prisma**, and **MySQL**. Includes auth, RBAC, drag‑and‑drop, optimistic UI, and a clean deployment story. This README is written like a Solutions Engineer: it covers the *what*, *why*, and *how*—plus onboarding steps, architecture, and a demo script.

---

## ✨ Highlights

* **Modern stack**: Next.js 14 (App Router), React Server Actions, TypeScript, Tailwind.
* **Data**: Prisma ORM + MySQL (Aiven/PlanetScale/local), Zod for validation.
* **Auth**: Clerk (email/password, OAuth) with optional orgs/teams.
* **RBAC**: Owner/Admin/Member roles per board.
* **Drag & drop**: React DnD with keyboard‑accessible reordering.
* **Optimistic UI**: Latency‑tolerant create/move/update for lists & cards.
* **Observability**: Request metrics + structured logging hooks.
* **CI/CD**: GitHub Actions for lint, type‑check, build, and Prisma migrations.
* **One‑click deploy**: Vercel; environment‑driven configuration.

> **Demo tagline**: Create boards, organize lists, drag cards, and collaborate—all in under 60 seconds.

---

## 🧭 Table of Contents

1. [Architecture](#-architecture)
2. [Domain Model](#-domain-model)
3. [User Stories](#-user-stories)
4. [Screens & Flows](#-screens--flows)
5. [Tech Stack](#-tech-stack)
6. [Getting Started](#-getting-started)
7. [Environment Variables](#-environment-variables)
8. [Database & Migrations](#-database--migrations)
9. [Running Locally](#-running-locally)
10. [Testing](#-testing)
11. [CI/CD](#-cicd)
12. [Security & Compliance](#-security--compliance)
13. [Performance](#-performance)
14. [Accessibility](#-accessibility)
15. [Observability](#-observability)
16. [Demo Script (SE)](#-demo-script-se)
17. [Troubleshooting](#-troubleshooting)
18. [Roadmap](#-roadmap)
19. [FAQ](#-faq)
20. [License](#-license)

---

## 🏗 Architecture

**Front end**

* Next.js App Router with server components
* Client components for DnD interactions
* Tailwind + Radix UI primitives (or shadcn/ui) for composable UX

**Back end**

* Route handlers (`app/api/*`) for CRUD (boards, lists, cards, comments, labels)
* Server Actions for low‑latency mutations with optimistic UI
* Prisma for DB access; Zod for runtime validation

**Auth & Org**

* Clerk for user auth; optional Organizations (tenant‑aware boards)
* Middleware to inject session and org context

**Storage (optional)**

* Uploadthing / S3 for attachments

**Infra**

* Vercel for hosting; Aiven/PlanetScale for MySQL
* GitHub Actions for CI

```
┌──────────────────────────────────┐
│            Browser               │
│  Next.js (RSC + client comps)    │
│  React DnD  | Tailwind | Clerk   │
└─────────────▲───────────┬────────┘
              │           │
              │ Server Actions / API Routes
              │           │
┌─────────────┴───────────▼────────┐
│           Next.js Server          │
│   Validation (Zod)  |  RBAC       │
│   Prisma Client      |  Logging    │
└─────────────▲───────────┬─────────┘
              │           │
              │           │ Prisma
              │           │
        ┌─────┴──────┐  ┌─▼──────────┐
        │   MySQL    │  │ Object/S3  │
        │  (Aiven)   │  │ Attachments│
        └────────────┘  └────────────┘
```

---

## 🗂 Domain Model

* **User** ← Clerk user id
* **Organization** (optional) ← Clerk org id
* **Board**: title, ownerOrgId/ownerUserId, visibility
* **Membership**: userId, boardId, role (owner/admin/member)
* **List**: title, order, boardId
* **Card**: title, description, order, listId, dueDate, labels[]
* **Comment**: body, cardId, authorId
* **Label**: name, color, boardId
* **Attachment** (optional): url, fileMeta, cardId

> Ordering is stable with integer indices. Moves recalc neighbor indices server‑side for consistency.

---

## ✅ User Stories

* As a user, I can sign in and create a board.
* As a board owner, I can invite members and set roles.
* As a member, I can create lists and cards, drag to reorder or move across lists.
* I can add descriptions, due dates, labels, and comments to a card.
* I can search/filter cards by text/label/due.

---

## 🖼 Screens & Flows

* **Boards**: create, rename, archive
* **Board View**: lists grid, card tiles, drag‑and‑drop
* **Card Drawer/Modal**: description, activity, labels, due date, attachments
* **Invite Flow**: add member by email (Clerk user)

> Include screenshots or Loom walkthrough links here when available.

---

## 🧰 Tech Stack

* **App**: Next.js 14, React 18, TypeScript
* **UI**: Tailwind CSS, Radix/shadcn (buttons, dialogs, dropdowns)
* **DnD**: `react-dnd`
* **State**: React state + server actions; SWR for cache revalidation
* **Data**: Prisma, MySQL, Zod
* **Auth**: Clerk (JWT + middleware); optional orgs/teams
* **Testing**: Vitest / Jest, Testing Library, Playwright (e2e)
* **Quality**: ESLint, Prettier, TypeScript strict
* **CI**: GitHub Actions
* **Deploy**: Vercel

---

## 🚀 Getting Started

### Prerequisites

* Node 20+
* pnpm (recommended) or npm
* MySQL 8+ (local via Docker) or managed (Aiven/PlanetScale)
* Clerk account (if using hosted auth)

### Quickstart

```bash
# 1) Install deps
pnpm install

# 2) Copy env
cp .env.example .env

# 3) Set DATABASE_URL & Clerk keys in .env

# 4) Generate & migrate
pnpm prisma:generate
pnpm prisma:migrate

# 5) Seed (optional)
pnpm prisma:seed

# 6) Start
pnpm dev
```

**Package.json scripts (sample)**

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev --name init",
    "prisma:deploy": "prisma migrate deploy",
    "prisma:studio": "prisma studio",
    "prisma:seed": "ts-node prisma/seed.ts",
    "test": "vitest",
    "e2e": "playwright test"
  }
}
```

---

## 🔐 Environment Variables

Create `.env` (see `.env.example`).

```
# App
NEXT_PUBLIC_APP_URL=https://localhost:3000

# Database (MySQL)
DATABASE_URL="mysql://USER:PASSWORD@HOST:PORT/DBNAME?ssl-mode=REQUIRED"

# Clerk (Auth)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
# Optional orgs (set in Clerk dashboard)
# CLERK_ORGANIZATIONS_ENABLED=true

# Storage (optional)
S3_BUCKET=
S3_REGION=
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
```

> **Note**: If using **Aiven** or **PlanetScale**, paste the provided connection URL into `DATABASE_URL`. For local Docker MySQL, see below.

---

## 🗄 Database & Migrations

**Prisma schema** lives in `prisma/schema.prisma` and models Users/Boards/Lists/Cards, etc.

### Run MySQL locally (Docker)

```bash
docker run --name kanban-mysql -e MYSQL_ROOT_PASSWORD=pass -e MYSQL_DATABASE=kanban \
  -p 3306:3306 -d mysql:8
```

Set:

```
DATABASE_URL="mysql://root:pass@127.0.0.1:3306/kanban"
```

### Migrate & seed

```bash
pnpm prisma:generate
pnpm prisma:migrate
pnpm prisma:seed
```

Open **Prisma Studio** to inspect data:

```bash
pnpm prisma:studio
```

---

## ▶ Running Locally

```bash
pnpm dev
```

* App runs at `http://localhost:3000`
* Sign in with Clerk (or switch to passwordless dev users in seed)

**Build for prod**

```bash
pnpm build && pnpm start
```

---

## 🧪 Testing

* **Unit**: Vitest for utils, RBAC, sorting logic
* **Component**: React Testing Library for UI states
* **E2E**: Playwright for create board → add list → add/move card → edit card

Run all:

```bash
pnpm test
pnpm e2e
```

---

## 🔁 CI/CD

**GitHub Actions** workflow:

* Install, cache deps
* `pnpm lint` + `pnpm typecheck`
* `pnpm prisma:generate` + `pnpm build`
* (Optional) Spin up ephemeral MySQL for `pnpm test`
* On `main`: deploy to Vercel; run `prisma migrate deploy` on production DB

Store secrets in GitHub → Actions secrets.

---

## 🛡 Security & Compliance

* Role‑based access enforced server‑side per board
* Input validation via Zod on all mutations
* Session bound to Clerk JWT; middleware blocks unauthenticated routes
* OWASP: CSRF (same‑site cookies), XSS (React escapes), SSRF (no external fetch in user inputs)
* PII scope minimal; audit logs for admin actions (optional feature)

---

## ⚡ Performance

* Optimistic UI for card/list mutations
* RSC + route segment caching for board lists
* `useTransition` for non‑blocking UI updates
* Indexes on `(boardId, order)`, `(listId, order)`
* Avoid N+1 via Prisma `include` & batched fetches

---

## ♿ Accessibility

* Keyboard DnD fallbacks (Up/Down to reorder; Enter to pick/drop)
* ARIA roles for list → `list`, card → `listitem`
* Focus outlines & skip links
* Color‑contrast‑safe label palette

---

## 👀 Observability

* Request timing via `performance.now()` wrappers
* Structured logs (action, actorId, boardId, latencyMs)
* Hook points for Sentry (exceptions) and PostHog (feature usage)

---

## 🎤 Demo Script (SE)

**Goal**: Show business value in <5 minutes.

1. **Context (20s)**: “Teams need clarity. This Kanban gives instant visibility of work in progress.”
2. **Create Board (30s)**: New board → name “Q4 Campaign”.
3. **Add Lists (30s)**: To Do / In Progress / Done.
4. **Add Cards (45s)**: “Design Banners”, “Email Blasts”, “Landing Page Copy”.
5. **Move Cards (30s)**: Drag to reorder; show keyboard move.
6. **Card Details (60s)**: Open card → add description, due date, label; comment “Blocked by assets”.
7. **Collab (30s)**: Invite member; set role to Member; show RBAC.
8. **Wrap (20s)**: Metrics hook + audit log (optional). Call to action.

**Talking Points**

* Reduces status meetings by visualizing throughput
* Prioritization via drag‑and‑drop; due dates prevent slippage
* RBAC keeps boards secure in multi‑team orgs

---

## 🧭 Troubleshooting

* **Loom iframe blocked**: Loom uses `X-Frame-Options: deny`. Use Loom’s embed URL or link out instead of iframe where denied.
* **Clerk org slugs disabled**: Enable Organization slugs in Clerk dashboard, or remove org routes and use user‑scoped boards only.
* **Duplicate subscriptions appearing in Stripe Billing Portal**: Ensure a single Stripe customer per user across projects (not used in this app; general SE note).
* **Aiven “org taken”**: That’s app‑level naming, not DB. Clearing DB won’t change SaaS org slugs—rename in app or enable slugs.
* **Overlay taller than image**: Ensure parent is `relative` and overlay is `absolute inset-0` with matching `overflow-hidden` on wrapper.

---

## 🗺 Roadmap

* [ ] Real‑time presence (WebSockets)
* [ ] Activity feed & audit logs
* [ ] Attachments (S3)
* [ ] Board templates
* [ ] Subtasks & checklists
* [ ] Calendar view & iCal export
* [ ] Advanced filters and saved views
* [ ] Public board sharing (read‑only)

---

## ❓ FAQ

**Why MySQL?**

* Great with Prisma for ordered lists; PlanetScale/Aiven provide serverless‑friendly endpoints.

**Can I switch to Postgres?**

* Yes—update Prisma provider and run new migrations; ordering logic remains the same.

**Do I need Clerk?**

* You can swap for NextAuth; see `auth/` adapter interface.

---

## 📄 License

MIT. See `LICENSE`.

---

## 📎 Appendix

**File tree (excerpt)**

```
app/
  (marketing)/
  dashboard/
  api/
lib/
components/
prisma/
  schema.prisma
  seed.ts
```

**Seed data** creates a demo board with 3 lists and 6 cards for easy testing.

