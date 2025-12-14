# {{PROJECT_NAME}} – Mobile Project Initialisation (Expo / React Native)

_Last updated: {{DATE}}_

---

## 1. Project Overview

**Elevator pitch**  
{{SHORT_DESCRIPTION}}

**Target platforms**  
- iOS: {{MIN_IOS_VERSION}}+
- Android: {{MIN_ANDROID_VERSION}}+

**Primary users**  
- {{USER_TYPE_1}}
- {{USER_TYPE_2}}

**Core problems this solves**  
- {{PROBLEM_1}}
- {{PROBLEM_2}}

**v1 outcomes**  
1. {{OUTCOME_1}}
2. {{OUTCOME_2}}
3. {{OUTCOME_3}}

---

## 2. Tech Stack & Architecture Principles

### Framework & Runtime

- Expo + React Native
- expo-router for navigation (file-based routing)
- TypeScript (`strict: true`)

### UI

- React Native components
- (Optional) NativeWind or utility-first styling
- Shared UI under `components/ui`
- Non-feature shared UI under `components/common`

### State & Data Layer

- Server state: **React Query**
- Global state: none by default; add **Zustand** or **Redux Toolkit** only when needed
- Feature-level state lives inside `features/<feature>/hooks`

### Backend & Services

- Firebase (Auth, Firestore, Storage) via `core/firebase`
- Custom REST/GraphQL API via `core/api`
- All async data flows through React Query

### Platform / App-Wide Integrations (`core/`)

- Firebase wrappers
- API client
- React Query setup
- Zustand/Redux setup (optional)
- Analytics / tracking
- i18n
- Typed env variables

### Testing

- Unit/integration tests: **Jest** + React Native Testing Library
- E2E tests: **Maestro**
- Unit tests colocated next to code
- Shared test utils → `/tests`

---

## 3. Folder Structure (Mobile)

```txt
/
  app/                     # expo-router routes only

  core/                    # platform-level integrations
    query/                 # React Query setup
    state/                 # Zustand/RTK (optional)
    api/                   # API clients
    firebase/              # Firebase wrappers
    tracking/              # analytics
    i18n/                  # localisation
    env/                   # typed env helpers

  components/              # shared UI components
    ui/
    common/

  features/                # self-contained feature modules
    exampleFeature/
      components/
      hooks/
      utils/
      types.ts
      index.ts

  types/                   # shared domain types

  assets/                  # Expo assets
    images/
    icons/
    fonts/
    audio/
    video/

  tests/                   # Jest utilities, fixtures, mocks
    setup/
      jest.setup.ts
    fixtures/
      user.ts
    utils/
      renderWithProviders.tsx

  e2e/
    maestro/
      flows/
        smoke.yaml
        auth.yaml
      config.yaml
```

---

## 4. Folder Rules & Conventions

### `app/` (expo-router only)

- No business logic
- Minimal UI logic
- Routes delegate to feature components

### `core/` – Infrastructure Layer

- Only platform-level concerns
- No UI components
- No feature-specific code

### `components/`

- `ui/` → primitives
- `common/` → shared multi-feature UI
- Cannot import from `features/*`

### `features/`

A feature contains everything related to one domain area.

- UI
- Hooks
- Utils
- Types
- Public API (`index.ts`)

### `types/`

- Shared domain models (`User`, `AuthToken`, etc.)

### `assets/`

Organised static assets.

---

## 5. State Management & Data Fetching

### React Query (default server-state engine)

Pattern:

1. `core/api` defines raw API/fetch functions  
2. `features/<feature>/hooks` exposes React Query hooks  
3. Query keys are defined in each feature  

### Optional global state (Zustand/Redux)

Use sparingly for:

- session state
- global UI (theme/modals/etc.)
- device capability flags

---

## 6. Backend Integration

### Firebase (`core/firebase`)

```txt
core/firebase/
  client.ts
  auth.ts
  db.ts
  storage.ts
```

Rules:

- Only this folder imports Firebase SDK
- Features must use these wrappers, not Firebase directly

### Custom API (`core/api`)

- Base client
- Endpoints grouped by domain (user, auth, etc.)
- Features use API functions via React Query hooks

---

## 7. Environment Variables

`.env`, `.env.development`, `.env.production`

Typed access via:

```txt
core/env/index.ts
```

Rule:

- Never use `process.env` directly outside core/env

---

## 8. Testing Strategy

### Unit/Integration (Jest + RNTL)

- Tests colocated with code: `file.test.ts` or `file.test.tsx`
- Shared helpers in `tests/utils`
- Expect tests for:
  - feature hooks  
  - feature utils  
  - core/api functions  
  - screens with logic  

### E2E (Maestro)

```txt
e2e/maestro/
  flows/
    smoke.yaml
    auth.yaml
  config.yaml
```

Rules:

- Only test key flows  
- Keep selectors simple and stable  

---

## 9. AI Collaboration Guide

When working with AI, always include:

### Context

- “Expo + React Native + TS project”
- “expo-router navigation”
- “Architecture: core for infra, features for domain modules”
- “React Query for data fetching”
- “Firebase + custom API combo”

### Goal

Clear description of what you want to build/change.

### Relevant files & paths

Include only necessary files to reduce ambiguity.

### Constraints

- Follow folder architecture  
- Use TypeScript  
- Use React Query  
- Use core/firebase and core/api wrappers  
- Add/update colocated Jest tests  
- Update Maestro flow if main path changes  

### Required tests

Always request:

> “Include colocated Jest tests. If this affects core user flows, also update/create a Maestro flow.”

---

## 10. Quality Gates (CI/CD)

- Lint must pass  
- Type-check must pass  
- Jest tests must pass  
- Maestro flows must pass (optional)  
- iOS + Android builds succeed  

---

## 11. New Mobile Project Setup Checklist

- [ ] Init Expo project with TypeScript  
- [ ] Install expo-router and configure routes  
- [ ] Create folder structure  
- [ ] Implement core/query with QueryClient  
- [ ] Implement core/firebase and test connection  
- [ ] Create assets folders  
- [ ] Setup Jest + RNTL with `tests/setup/jest.setup.ts`  
- [ ] Add `renderWithProviders.tsx`  
- [ ] Add example feature module  
- [ ] Add Maestro with a smoke test  
- [ ] Commit `project-initialisation.mobile.md`  

---
