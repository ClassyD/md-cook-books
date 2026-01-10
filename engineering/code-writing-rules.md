Code Writing Rules & Guidelines

---

## Component Reusability Principle

**Default to shared components**: Always check if a component already exists in shared component directories before building a new one.

### Shared Components for Common UI

- ✅ Buttons, Cards, Inputs, Lists, Modals, etc. → Shared UI primitives
- ✅ Multi-feature shared components (e.g., Header, Footer, Navigation) → Shared common components
- ✅ These should be built once and reused across all features for consistency

### Feature-Specific Components Only When

- ❌ The component contains business logic specific to one feature
- ❌ The component is so unique to a feature that it won't be reused elsewhere
- ❌ The component requires feature-specific hooks or data structures that can't be abstracted

### Decision Process

1. Need a Button? → Check shared UI components first
2. Need a Card? → Check shared UI components first
3. Need a ProductCard? → If it's just a styled card, extend the shared Card component
4. Need a ProductCard with product-specific business logic? → Create in feature-specific components

**Rule**: When in doubt, start with a shared component. It's easier to move a component from feature-specific to shared later than to duplicate and maintain multiple versions.

---

## Screen-Level Hook Pattern

**Rule**: All business logic for a screen must live in a dedicated hook. The UI component should be primarily presentational.

### Pattern Structure

Each screen follows this pattern:

1. **Screen Route** (expo-router/Next.js route) - Minimal, just calls the hook and renders the view
2. **Screen Hook** (`use<ScreenName>Screen`) - Contains all business logic, data fetching, state management
3. **Screen View Component** (`<ScreenName>View`) - Pure presentational component

### What Goes Where

**In the Hook:**
- ✅ Data fetching (React Query hooks)
- ✅ State management (local state, derived state)
- ✅ Event handlers with business logic
- ✅ Side effects (navigation, analytics, etc.)
- ✅ Data transformations
- ✅ Validation logic
- ✅ Error handling

**In the UI Component:**
- ✅ Rendering logic
- ✅ Conditional rendering based on props
- ✅ Layout and styling
- ✅ Simple prop forwarding to child components
- ❌ No data fetching
- ❌ No business logic
- ❌ No side effects (except UI-only effects like animations)

### Hook Naming

- Screen hooks: `use<ScreenName>Screen` (e.g., `useHomeScreen`, `useProfileScreen`)
- Feature hooks: `use<FeatureName><Action>` (e.g., `useAuthLogin`, `useProductList`)

---

## useEffect Usage Rules

**Rule**: `useEffect` is an escape hatch from the React paradigm. Only use it to synchronize with external systems. Avoid using it for data transformations, state updates based on props/state, or handling user events.

### When NOT to Use useEffect

**❌ Don't use useEffect to transform data for rendering**
- Calculate derived values during render instead
- Example: Filtering a list, computing `fullName` from `firstName` + `lastName`
- **Why**: Causes unnecessary render passes and is inefficient

**❌ Don't use useEffect to update state based on props or state**
- Calculate values during rendering instead
- Example: Computing `fullName` from `firstName` and `lastName` state
- **Why**: Creates redundant state and causes cascading updates

**❌ Don't use useEffect to handle user events**
- Use event handlers directly
- Example: Sending POST requests, showing notifications on button clicks
- **Why**: Event handlers know exactly what happened; Effects don't

**❌ Don't use useEffect to cache expensive calculations**
- Use `useMemo` instead
- Example: Filtering large lists, expensive computations
- **Why**: `useMemo` is designed for this purpose and runs during render

**❌ Don't use useEffect to reset state when props change**
- Set state during rendering or use a `key` prop to reset entire component tree
- **Why**: More direct and avoids unnecessary Effect dependencies

### When TO Use useEffect

**✅ Use useEffect to synchronize with external systems**
- Non-React widgets (e.g., jQuery plugins)
- Browser APIs (e.g., `window.addEventListener`, `document.title`)
- Network subscriptions
- Third-party libraries that need cleanup

**✅ Use useEffect for data fetching**
- Synchronizing component state with server data
- Note: Modern frameworks provide better built-in data fetching mechanisms
- Always implement cleanup to avoid race conditions

**✅ Use useEffect for subscriptions**
- WebSocket connections
- Event listeners
- Timers/intervals
- Always return cleanup functions

### Common Patterns

**Instead of this:**
```typescript
// ❌ Bad: Using useEffect to compute derived state
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

**Do this:**
```typescript
// ✅ Good: Calculate during render
const fullName = `${firstName} ${lastName}`;
```

**Instead of this:**
```typescript
// ❌ Bad: Using useEffect to filter data
const [visibleTodos, setVisibleTodos] = useState([]);
useEffect(() => {
  setVisibleTodos(todos.filter(todo => !todo.completed));
}, [todos]);
```

**Do this:**
```typescript
// ✅ Good: Calculate during render (or useMemo if expensive)
const visibleTodos = useMemo(
  () => todos.filter(todo => !todo.completed),
  [todos]
);
```

**Instead of this:**
```typescript
// ❌ Bad: Using useEffect for user actions
useEffect(() => {
  if (shouldSubmit) {
    fetch('/api/buy', { method: 'POST' });
  }
}, [shouldSubmit]);
```

**Do this:**
```typescript
// ✅ Good: Handle in event handler
function handleBuyClick() {
  fetch('/api/buy', { method: 'POST' });
  showNotification('Purchase complete!');
}
```

### Data Fetching with useEffect

When fetching data with `useEffect`, always:
1. **Implement cleanup** to ignore stale responses (prevent race conditions)
2. **Handle loading and error states**
3. **Consider using React Query or similar** for better caching and state management

```typescript
// ✅ Good: Data fetching with cleanup
useEffect(() => {
  let ignore = false;
  
  fetchResults(query, page).then(json => {
    if (!ignore) {
      setResults(json);
    }
  });
  
  return () => {
    ignore = true;
  };
}, [query, page]);
```

**Reference**: [React: You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)

---

## Test File Placement

**Rule**: Tests are colocated with the source code they test.

### Test File Locations

- `features/home/hooks/useHomeScreen.ts` → `features/home/hooks/useHomeScreen.test.ts`
- `features/home/components/HomeScreenView.tsx` → `features/home/components/HomeScreenView.test.tsx`
- `core/api/user.ts` → `core/api/user.test.ts`
- `features/auth/utils/validation.ts` → `features/auth/utils/validation.test.ts`

### Testing Screen Patterns

When testing screens:

1. **Test the hook** – Test all business logic
2. **Test the view component** – Test rendering and prop handling
3. **Test integration** – Test the screen route renders correctly

### Testing Libraries

**Rule**: Use Testing Library for all React and React Native tests. Prefer testing behavior and user-visible outcomes over implementation details.

- Web (React): Use `@testing-library/react`
- React Native: Use `@testing-library/react-native`

### Testing Guidelines

**Test behavior, not implementation**

- ✅ Assert what the user sees and can do (text, buttons, navigation, error messages)
- ✅ Assert observable side effects (e.g., API calls via mocks, navigation via mocked router)
- ❌ Don’t assert internal state variables or private functions
- ❌ Don’t couple tests to specific hook implementations or component internals

**Use accessibility-first queries**

- Prefer `getByRole`, `getByLabelText`, `getByPlaceholderText`, `getByText`
- Use `getByTestId` only as a last resort when no semantic query fits

**Use Testing Library patterns**

- Use `render(...)` from the relevant Testing Library package
- Use `screen` and `userEvent` (web) or `fireEvent` / similar user interaction helpers (native) to simulate user interactions
- Avoid manual DOM/instance access (e.g., `container.querySelector`) unless absolutely necessary

**Align with Screen-Level Hook Pattern**

- Test hooks with `renderHook` (or dedicated hook testing utilities)
- Test views with Testing Library, focusing on rendered output and interactions
- Write integration tests at the route/screen level that exercise the full UI behavior end-to-end

---

## TypeScript Guidelines

- **Always use TypeScript** with `strict: true`
- **Type everything** – Avoid `any`, use proper types or `unknown`
- **Use interfaces for props** – Clear, well-documented prop types
- **Export types** – Make types reusable when they might be used elsewhere

---

## Code Organization

### Import Order

1. External dependencies (React, libraries)
2. Internal core modules (`core/`)
3. Feature modules (`features/`)
4. Shared components (`components/`)
5. Types
6. Utils and helpers
7. Relative imports

### File Naming

- Components: `PascalCase.tsx` (e.g., `Button.tsx`, `HomeScreenView.tsx`)
- Hooks: `camelCase.ts` with `use` prefix (e.g., `useHomeScreen.ts`)
- Utils: `camelCase.ts` (e.g., `formatDate.ts`, `validation.ts`)
- Types: `camelCase.ts` or `types.ts` (e.g., `userTypes.ts`)
- Tests: `file.test.ts` or `file.test.tsx` (colocated with source)

---

## General Principles

1. **DRY (Don't Repeat Yourself)** – Extract common logic into reusable functions/components
2. **Single Responsibility** – Each function/component should do one thing well
3. **Composition over Inheritance** – Prefer composing smaller pieces
4. **Explicit over Implicit** – Make code intentions clear
5. **Fail Fast** – Validate inputs early, throw clear errors
6. **Consistency** – Follow established patterns in the codebase

---

