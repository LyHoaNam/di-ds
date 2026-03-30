# 2. Pattern Catalog

[← Back to Index](frontend-engineering-framework.md)

---

## 2.1 Component Patterns

### Presentational vs. Container Components

```
┌──────────────────┐     ┌──────────────────┐
│   Container      │     │  Presentational   │
│  (Smart)         │────▶│  (Dumb)           │
│                  │     │                   │
│ • Fetches data   │     │ • Receives props  │
│ • Manages state  │     │ • Renders UI      │
│ • Side effects   │     │ • No side effects │
│ • Business logic │     │ • Reusable        │
└──────────────────┘     └──────────────────┘
```

### Compound Components

Components that share implicit state and work together as a cohesive unit.

```jsx
<Select>
  <Select.Trigger />
  <Select.Options>
    <Select.Option value="a">Alpha</Select.Option>
    <Select.Option value="b">Beta</Select.Option>
  </Select.Options>
</Select>
```

Use cases: Tabs, Accordions, Menus, Form Groups, Dropdowns.

### Render Props / Scoped Slots

Delegate rendering control to the consumer while the component manages logic.

```jsx
<DataFetcher url="/api/users">
  {({ data, loading, error }) => (
    loading ? <Spinner /> : <UserList users={data} />
  )}
</DataFetcher>
```

### Higher-Order Components (HOC)

A function that takes a component and returns an enhanced component.

```
withAuth(Component)  → Adds authentication gate
withTheme(Component) → Injects theme context
withLogger(Component) → Adds lifecycle logging
```

**Modern alternative**: Custom hooks / composables are now preferred over HOCs.

### Headless Components (Renderless)

Components that encapsulate behavior with zero UI opinions. The consumer provides all rendering.

```
useCombobox()   → Manages keyboard nav, filtering, selection
useDialog()     → Manages open/close, focus trap, escape key
usePagination() → Manages page state, boundaries, navigation
```

Libraries: Headless UI, Radix, Downshift, TanStack Table.

### Provider Pattern

Share global/cross-cutting data without prop drilling.

```
<ThemeProvider>
  <AuthProvider>
    <RouterProvider>
      <App />
    </RouterProvider>
  </AuthProvider>
</ThemeProvider>
```

**Warning**: Avoid a single monolithic provider. Split contexts by domain to prevent unnecessary re-renders.

### Controlled vs. Uncontrolled Components

| Aspect | Controlled | Uncontrolled |
|--------|-----------|-------------|
| State owner | Parent component | DOM / internal ref |
| Data flow | Props + onChange | Ref-based access |
| Validation | Real-time | On submit |
| Use case | Forms with complex logic | Simple forms, file inputs |

---

## 2.2 State Management Patterns

### Local Component State

- `useState`, `useReducer` (React), `ref()`, `reactive()` (Vue), class properties (Angular).
- Default choice — lift state only when necessary.

### Lifted State / Prop Drilling

- Move shared state to the nearest common ancestor.
- Acceptable for <3 levels of depth. Beyond that, use context or state management.

### Global Store (Flux / Redux Pattern)

```
┌────────┐  dispatch  ┌────────────┐  reduce  ┌───────┐  select  ┌──────┐
│ Action │──────────▶ │ Dispatcher │────────▶ │ Store │────────▶ │ View │
└────────┘            └────────────┘          └───────┘          └──────┘
     ▲                                                               │
     └───────────────────────────────────────────────────────────────┘
```

Libraries: Redux (Toolkit), Zustand, Pinia, NgRx, MobX.

### Atomic State

State is broken into independent atoms that can be composed.

```
const userAtom = atom(null)
const themeAtom = atom('light')
const derivedAtom = computed(() => ...)  // derived state
```

Libraries: Jotai, Recoil, Nanostores.

### Server State (Cache-First)

Treats remote data as a cache, not as application state.

```
┌─────────┐  fetch/mutate  ┌──────────┐  hydrate  ┌──────┐
│  Server │◀──────────────▶│  Cache   │─────────▶ │  UI  │
└─────────┘                └──────────┘           └──────┘
                           (stale-while-revalidate)
```

Libraries: TanStack Query, SWR, RTK Query, Apollo Client.

Key concepts: caching, background refetching, optimistic updates, pagination, infinite scroll.

### State Machines / Statecharts

Model complex UI flows as finite state machines.

```
idle → loading → success
                → error → retry → loading
```

Eliminates impossible states. Libraries: XState, Robot.

### URL as State

- The URL is the single source of truth for navigation-related state.
- Search params, hash fragments, and path segments encode application state.
- Enables deep linking, sharing, and back/forward navigation.

---

## 2.3 Data Fetching Patterns

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Fetch-on-render** | Component mounts, then fetches | Simple cases, client-only apps |
| **Fetch-then-render** | Fetch completes before route renders | SSR, loaders (Remix, Next.js) |
| **Render-as-you-fetch** | Fetch starts early, component renders in parallel | Suspense-based architectures |
| **Prefetching** | Fetch on hover/focus before navigation | Predictive UX optimization |
| **Optimistic updates** | UI updates instantly, rolls back on failure | Mutations where latency matters |
| **Polling** | Repeated fetches at interval | Dashboards, status pages |
| **WebSocket / SSE** | Server pushes updates to client | Real-time: chat, notifications, collaboration |

---

## 2.4 Rendering Patterns

| Pattern | Abbreviation | Where it Runs | When HTML is Generated |
|---------|-------------|---------------|----------------------|
| Client-Side Rendering | CSR | Browser | On each page load |
| Server-Side Rendering | SSR | Server | On each request |
| Static Site Generation | SSG | Build time | At build time |
| Incremental Static Regeneration | ISR | Server (on-demand) | After stale timeout |
| Streaming SSR | — | Server → Browser | Chunks sent as ready |
| Partial Hydration | — | Server + Browser | Only interactive parts hydrate |
| Islands Architecture | — | Server + Browser | Static shell + interactive islands |
| Resumability | — | Server + Browser | Zero hydration cost (Qwik) |

---

## 2.5 Cross-Cutting Patterns

### Error Handling Strategy

```
┌──────────────────────────────────────────────┐
│           Global Error Boundary               │
│  ┌────────────────────────────────────────┐  │
│  │       Route-Level Error Boundary       │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │  Feature-Level Error Boundary    │  │  │
│  │  │  ┌───────────────────────────┐   │  │  │
│  │  │  │  Component try/catch      │   │  │  │
│  │  │  └───────────────────────────┘   │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

Layers: component-level → feature-level → route-level → global fallback.

### Authentication / Authorization Patterns

- **Route guards**: Protect routes based on auth state.
- **Permission-based rendering**: Show/hide UI elements based on roles.
- **Token refresh**: Silent refresh with interceptors.
- **Session management**: Secure cookie-based or token-based flows.

### Internationalization (i18n)

- Extract strings into locale files (JSON, ICU format).
- Handle pluralization, date/number formatting, RTL layouts.
- Lazy-load locale bundles to minimize payload.

### Feature Flags

- Decouple deployment from release.
- Gradual rollouts, A/B testing, kill switches.
- Libraries: LaunchDarkly, Unleash, Flagsmith; or a simple JSON config with a context provider.

---

Previous: [← Principles Framework](01-principles-framework.md) · Next: [Architecture Styles →](03-architecture-styles.md)
