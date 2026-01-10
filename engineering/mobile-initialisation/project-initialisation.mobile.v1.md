Mobile Project Initialisation (Expo / React Native)

---

## Prerequisites

- **Always use TypeScript** – All projects must be TypeScript with `strict: true`
- **Use latest versions** – Always use the latest stable versions of all tools, frameworks, and dependencies unless a specific version is explicitly mentioned
- **Check compatibility** – Ensure compatibility between major dependencies (e.g., React Native/Expo, React Query with its React version) when determining versions
- **LLM responsibility** – The LLM should determine the latest available versions and appropriate installation commands at the time of project initialization
- **Code Writing Rules** – The LLM must ensure the code-writing-rules.md file is available as a Cursor rule. See Section 11 for setup instructions.

---

## 1. Tech Stack & Architecture Principles

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

## 2. Folder Structure (Mobile)

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

## 3. Folder Rules & Conventions

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

## 4. State Management & Data Fetching

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

## 5. Backend Integration

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

## 6. Environment Variables

`.env`, `.env.development`, `.env.production`

Typed access via:

```txt
core/env/index.ts
```

Rule:

- Never use `process.env` directly outside core/env

---

## 7. Testing Strategy

### Unit/Integration (Jest + RNTL)

- Tests colocated with code: `file.test.ts` or `file.test.tsx`
- **Test file location**: Place test files next to the source file they test
  - `features/home/hooks/useHomeScreen.ts` → `features/home/hooks/useHomeScreen.test.ts`
  - `features/home/components/HomeScreenView.tsx` → `features/home/components/HomeScreenView.test.tsx`
  - `core/api/user.ts` → `core/api/user.test.ts`
  - `features/auth/utils/validation.ts` → `features/auth/utils/validation.test.ts`
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

## 8. Development Guide: Components & Screen Patterns

### Screen-Level Hook Pattern

**Rule**: All business logic for a screen must live in a dedicated hook. The UI component should be primarily presentational.

#### Pattern Structure

Each screen follows this pattern:

```typescript
// app/home.tsx (expo-router route)
import { useHomeScreen } from '@/features/home/hooks/useHomeScreen';

export default function HomeScreen() {
  const homeScreen = useHomeScreen();
  
  return <HomeScreenView {...homeScreen} />;
}
```

```typescript
// features/home/hooks/useHomeScreen.ts
export function useHomeScreen() {
  // All business logic lives here
  const { data, isLoading } = useHomeData();
  const { handleRefresh, handleItemPress } = useHomeActions();
  
  // Can compose other hooks
  const navigation = useNavigation();
  const analytics = useAnalytics();
  
  return {
    // Expose only what the UI needs
    data,
    isLoading,
    onRefresh: handleRefresh,
    onItemPress: handleItemPress,
  };
}
```

```typescript
// features/home/components/HomeScreenView.tsx
interface HomeScreenViewProps {
  data: HomeData[];
  isLoading: boolean;
  onRefresh: () => void;
  onItemPress: (id: string) => void;
}

export function HomeScreenView({ data, isLoading, onRefresh, onItemPress }: HomeScreenViewProps) {
  // Pure presentation logic only
  // No business logic, no data fetching, no side effects
  return (
    <ScrollView refreshControl={<RefreshControl onRefresh={onRefresh} />}>
      {isLoading ? <LoadingSpinner /> : <DataList data={data} onItemPress={onItemPress} />}
    </ScrollView>
  );
}
```

#### Rules

1. **Screen hook naming**: `use<ScreenName>Screen` (e.g., `useHomeScreen`, `useProfileScreen`)
2. **Hook location**: Screen hooks live in `features/<feature>/hooks/` or `app/<screen>/hooks/` if screen-specific
3. **Business logic**: All data fetching, state management, side effects, and business rules go in the hook
4. **UI components**: Should be pure presentational components that receive props and render
5. **Hook composition**: Screen hooks can compose other hooks (React Query hooks, feature hooks, utility hooks)
6. **Exposed interface**: The hook should expose a clean, typed interface of only what the UI needs

#### What Goes Where

**In the Hook (`useHomeScreen`):**
- ✅ Data fetching (React Query hooks)
- ✅ State management (local state, derived state)
- ✅ Event handlers with business logic
- ✅ Side effects (navigation, analytics, etc.)
- ✅ Data transformations
- ✅ Validation logic
- ✅ Error handling

**In the UI Component (`HomeScreenView`):**
- ✅ Rendering logic
- ✅ Conditional rendering based on props
- ✅ Layout and styling
- ✅ Simple prop forwarding to child components
- ❌ No data fetching
- ❌ No business logic
- ❌ No side effects (except UI-only effects like animations)

### Component Writing Guidelines

#### Component Reusability Principle

**Default to shared components**: Always check if a component already exists in `components/ui/` or `components/common/` before building a new one.

**Shared components for common UI:**
- ✅ Buttons, Cards, Inputs, Lists, Modals, etc. → `components/ui/`
- ✅ Multi-feature shared components (e.g., Header, Footer, Navigation) → `components/common/`
- ✅ These should be built once and reused across all features for consistency

**Feature-specific components only when:**
- ❌ The component contains business logic specific to one feature
- ❌ The component is so unique to a feature that it won't be reused elsewhere
- ❌ The component requires feature-specific hooks or data structures that can't be abstracted

**Decision process:**
1. Need a Button? → Check `components/ui/Button.tsx` first
2. Need a Card? → Check `components/ui/Card.tsx` first
3. Need a ProductCard? → If it's just a styled card, extend `components/ui/Card.tsx`
4. Need a ProductCard with product-specific business logic? → `features/products/components/ProductCard.tsx`

**Rule**: When in doubt, start with a shared component. It's easier to move a component from `features/` to `components/` later than to duplicate and maintain multiple versions.

#### Shared Components (`components/ui` and `components/common`)

- **Pure presentational components**
- **Reusable across features**
- **Well-typed props interfaces**
- **No feature-specific logic**
- **Can accept render props or children for flexibility**

#### Feature Components (`features/<feature>/components`)

- **Feature-specific UI components**
- **Can use feature hooks internally if needed**
- **Still prefer presentational when possible**
- **Can be composed by screen-level hooks**

#### Example: Reusable Component

```typescript
// components/ui/Button.tsx
interface ButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

export function Button({ title, onPress, variant = 'primary', disabled }: ButtonProps) {
  return (
    <Pressable onPress={onPress} disabled={disabled} style={[styles.base, styles[variant]]}>
      <Text>{title}</Text>
    </Pressable>
  );
}
```

### Testing Screen Patterns

When testing screens:

1. **Test the hook** (`useHomeScreen.test.ts`) – Test all business logic
2. **Test the view component** (`HomeScreenView.test.tsx`) – Test rendering and prop handling
3. **Test integration** – Test the screen route renders correctly

```typescript
// useHomeScreen.test.ts
describe('useHomeScreen', () => {
  it('fetches and returns home data', () => {
    const { result } = renderHook(() => useHomeScreen());
    expect(result.current.data).toBeDefined();
  });
  
  it('handles refresh correctly', () => {
    const { result } = renderHook(() => useHomeScreen());
    act(() => {
      result.current.onRefresh();
    });
    // Assert refresh behavior
  });
});
```

---

## 9. Quality Gates (CI/CD)

- Lint must pass  
- Type-check must pass  
- Jest tests must pass  
- Maestro flows must pass (optional)  
- iOS + Android builds succeed  

---

## 10. New Mobile Project Setup Checklist

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

## 11. Optional Tools & LLM Instructions

For any tool, library, or setup step described as **optional** in this document (e.g., NativeWind, Zustand/Redux, utility-first styling), the LLM should:

1. **Ask the user** whether the optional tool should be added or initialized before proceeding
2. **Not assume** that optional tools should be included by default
3. **Wait for confirmation** before installing dependencies or creating configuration files for optional tools

Examples of optional tools mentioned in this document:
- NativeWind or utility-first styling (Section 1)
- Zustand or Redux Toolkit for global state (Section 1, Section 4)
- Zustand/Redux setup in `core/state/` (Section 1, Section 2)

### Code Writing Rules File Setup

**Rule**: The LLM must ensure that the code-writing-rules.md file is available as a Cursor rule at `.cursor/rules/RULE.md`. This file contains essential guidelines for:
- Component reusability principles
- Screen-level hook patterns
- useEffect usage rules
- Test file placement
- TypeScript guidelines
- Code organization

**Setup Process:**

1. **Check if the rule exists**: First, check if `.cursor/rules/RULE.md` already exists in the project
2. **If it doesn't exist, create it**: The LLM should create the rule file directly in `.cursor/rules/`:
   ```
   .cursor/rules/
     RULE.md
   ```
3. **Fetch the content**: Attempt to fetch the content from the GitHub repository:
   - **GitHub Repository**: `https://github.com/ClassyD/md-cook-books`
   - **Raw File URL**: `https://raw.githubusercontent.com/ClassyD/md-cook-books/main/engineering/code-writing-rules.md`
4. **If GitHub access fails**: If the LLM cannot access the GitHub URL or fetch the file:
   - **Ask the user** to provide the code-writing-rules.md file
   - Request either:
     - The raw content of the file
     - Or ask the user to manually add it to `.cursor/rules/RULE.md`
5. **Create the RULE.md file**: Once the content is available, create the file with:
   - Frontmatter metadata (optional, can be added later via Cursor settings):
     ```markdown
     ---
     description: "Code writing rules and guidelines for React/React Native development"
     alwaysApply: true
     ---
     ```
   - The full content from the code-writing-rules.md file

**Reference**: [Cursor Rules Documentation](https://cursor.com/docs/context/rules#rule-folder-structure)

---

