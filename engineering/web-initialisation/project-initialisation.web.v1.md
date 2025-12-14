# {{PROJECT_NAME}} – Web Project Initialisation (Next.js)

_Last updated: {{DATE}}_

---

## 1. Project Overview

**Elevator pitch**  
{{SHORT_DESCRIPTION}}

**Primary users**  
- {{USER_TYPE_1}}
- {{USER_TYPE_2}}

**Problems this solves**  
- {{PROBLEM_1}}
- {{PROBLEM_2}}

**v1 outcomes**  
1. {{OUTCOME_1}}
2. {{OUTCOME_2}}
3. {{OUTCOME_3}}

---

## 2. Tech Stack & Architecture Principles

### Framework

- Next.js (App Router)
- TypeScript (`strict: true`)
- React 18 (server components default)

### UI

- Tailwind CSS
- shadcn/ui
- Shared UI under `components/ui`, non-feature shared UI under `components/common`

### State & Data Layer

- Server state: **React Query**
- Global app/UI state: **Zustand** (if needed)
- Feature-level state: colocated inside feature (`features/<feature>/hooks`)

### API & Backend Services

- Firebase (Auth, Firestore, Storage) wrapped in `core/firebase`
- HTTP/GraphQL clients in `core/api`
- Server Actions:
  - Feature-specific → `features/<feature>/serverActions`
  - Shared → `serverActions/`

### Platform / App-Wide Integrations (`core/`)

- Firebase
- React Query setup
- Zustand store setup
- Analytics / tracking
- i18n
- Typed environment variables
- API clients

### Testing

- Unit/integration tests: **Jest** + React Testing Library
- E2E tests: **Playwright**
- Unit tests colocated next to code
- Shared test utils → `/tests`
- Only thin smoke/critical path tests for e2e

---

## 3. Folder Structure (Web)

```txt
/
  public/                 # static assets
  app/                    # Next.js routes only (no business logic)

  core/                   # platform-level integrations → the "infrastructure layer"
    state/                # Zustand store setup
    query/                # React Query setup
    api/                  # API clients (REST/GraphQL)
    firebase/             # Firebase init + service wrappers
    tracking/             # analytics, logging
    i18n/                 # localisation
    env/                  # typed env variable access

  components/             # shared UI components
    ui/                   # shadcn primitives
    common/               # non-feature shared components

  features/               # self-contained feature modules
    exampleFeature/
      components/
      hooks/
      serverActions/
      utils/
      types.ts
      index.ts            # public API of the feature

  serverActions/          # cross-feature shared server actions
    auth/
    users/
    utils/

  types/                  # shared domain types/models
    index.ts

  tests/                  # Jest utilities, fixtures, mocks
    setup/
      jest.setup.ts
      msw.server.ts       # optional
    fixtures/
      user.ts
    utils/
      renderWithProviders.tsx

  e2e/                    # Playwright tests
    tests/
      smoke.spec.ts
      auth.spec.ts
    playwright.config.ts
```

---

## 4. Folder Rules & Conventions

### `app/` (ROUTES ONLY)

- No business logic.
- No API calls.
- Only:
  - Layouts
  - Pages
  - Route handlers
  - Providers (if needed)

---

### `core/` – THE INFRASTRUCTURE LAYER

Purpose: centralise integrations and cross-app concerns.

```txt
core/
  state/        → Zustand store
  query/        → React Query config + provider
  firebase/     → Firebase init + wrappers
  api/          → fetch/axios/GraphQL clients
  tracking/     → analytics wrappers
  i18n/         → localisation
  env/          → typed env helpers
```

**Rules**

- No React components here.
- No feature logic here.
- Everything must be reusable across the entire app.

---

### `components/`

```txt
components/
  ui/           → shadcn (pure UI primitives)
  common/       → shared reusable UI pieces
```

**Rules**

- Must not import from `features/*`.

---

### `features/`

```txt
features/
  <featureName>/
    components/       # UI for this feature only
    hooks/            # feature-specific logic (React Query hooks)
    serverActions/    # feature-only server actions
    utils/            # feature-only helper functions
    types.ts          # feature-only types
    index.ts          # controlled public API
```

**Rules**

- A feature owns its own UI, logic, hooks, types, and server actions.
- If more than one feature needs the same server action → move that action to `/serverActions`.

---

### `serverActions/` – shared server actions

- Used for cross-feature logic (auth, users, generic utilities).
- Must not import UI.
- Should import Firebase via `core/firebase`.

---

### `types/`

- Domain-level types shared across features (e.g. `User`, `Uuid`, `PaginatedResult`).
- Feature-specific types stay in the feature folder.

---

## 5. State Management & Data Fetching

### Global state – Zustand (`core/state`)

- Only used for:
  - session / auth state
  - UI-level global toggles (theme, drawer, modals)
  - device-specific flags if necessary

### Server state – React Query (`core/query`)

Pattern:

1. Low-level fetchers live in `core/api`.
2. Feature-level hooks wrap the fetchers:
   - `features/<feature>/hooks/useSomething.ts`
3. Caching & invalidation use feature-defined key factories.

**Example**

```ts
// features/user/hooks/useUserQuery.ts
import { useQuery } from "@tanstack/react-query";
import { getUser } from "@/core/api/user";

export const userKeys = {
  detail: (id: string) => ["user", id] as const,
};

export function useUserQuery(id: string) {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => getUser(id),
  });
}
```

---

## 6. Firebase Integration

Location:

```txt
core/firebase/
  client.ts       # initialises the app (singleton)
  auth.ts         # thin wrappers for auth
  db.ts           # Firestore helpers
  storage.ts      # Storage helpers
```

**Rules**

- Only core/firebase imports from Firebase SDK.
- Features must NOT import Firebase directly.
- For server actions, use the wrappers.

---

## 7. Environment Variables

Files:

```txt
.env.local
.env.development.local
.env.production.local
```

Typed access via:

```txt
core/env/
  index.ts
```

**Rule**  
Never use `process.env` directly outside `core/env`.

---

## 8. Testing Strategy

### 8.1 Unit & Integration Tests (Jest)

Tools:

- Jest
- @testing-library/react
- @testing-library/dom
- MSW (optional but recommended for mocking API/Firebase wrappers)

**Location**

- Tests colocated with code:
  - `file.ts` → `file.test.ts`
  - `file.tsx` → `file.test.tsx`

**Expectations**

- Every non-trivial piece of logic must have tests.
- Hooks must test:
  - success
  - loading
  - error paths
- High coverage on:
  - `core/api`
  - `core/firebase`
  - `features/*/hooks`
  - `features/*/utils`

### 8.2 Shared Test Utilities (`/tests`)

```txt
tests/
  setup/
    jest.setup.ts
    msw.server.ts     # if using MSW
  utils/
    renderWithProviders.tsx
  fixtures/
    user.ts
```

### 8.3 E2E Tests (Playwright)

Purpose: **thin smoke + core flows only**.

Location:

```txt
e2e/
  tests/
    smoke.spec.ts     # app boots, basic UI loads
    auth.spec.ts      # login/logout flow
  playwright.config.ts
```

**Rules**

- Keep E2E small, stable, easy to maintain.
- Avoid over-testing UI details.

---

## 9. AI Collaboration Guide

To ensure high-quality output from AI during development:

### Always provide:

1. **Context**
   - This is a Next.js + TypeScript project.
   - Architecture uses `core/`, `features/`, React Query, Zustand, Firebase wrappers.

2. **Goal**
   - What you're adding/changing.

3. **File paths**
   - Include only relevant files.

4. **Constraints**
   - "Use TypeScript"
   - "Follow my feature-based architecture"
   - "Put server actions in features/<feature>/serverActions"
   - "Use React Query for server data"
   - "Use core/firebase wrappers, never Firebase directly"

5. **Testing requirement**
   - Always request:
     - colocated Jest tests for any new or changed logic
     - update E2E tests if core flows change

### Example prompt

> I’m working on `features/profile` in a Next.js + TS project with React Query + Firebase (in core/firebase).  
> I need:  
> - a React Query hook `useProfileQuery(id)`  
> - a server action `updateProfile`  
> - colocated Jest tests for both  
> - and update `e2e/tests/auth.spec.ts` if needed  
> Here are my relevant files.  
> Follow the project architecture.

---

## 10. Quality Gates (CI/CD)

- Lint must pass
- Type-check must pass
- Jest tests must pass
- E2E tests must pass

---

## 11. New Project Setup Checklist

- [ ] Scaffold Next.js with TypeScript
- [ ] Install Tailwind + shadcn
- [ ] Create the folder structure
- [ ] Implement `core/query`, `core/state`, `core/firebase`
- [ ] Add `tests/setup/jest.setup.ts`
- [ ] Add example test in `features/example`
- [ ] Configure Playwright + first smoke test
- [ ] Add env handling via `core/env`
- [ ] Create first feature module

---
