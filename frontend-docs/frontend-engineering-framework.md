# Frontend Engineering Knowledge Framework

> A comprehensive reference and learning roadmap for frontend engineers — from foundational principles to advanced architecture.

---

## Table of Contents

1. [Principles Framework](#1-principles-framework)
2. [Pattern Catalog](#2-pattern-catalog)
3. [Architecture Styles](#3-architecture-styles)
4. [Performance Architecture](#4-performance-architecture)
5. [Team Standards](#5-team-standards)
6. [Suggested Learning Roadmap by Career Level](#6-suggested-learning-roadmap-by-career-level)

---

## 1. Principles Framework

### 1.1 Core Software Design Principles

| Principle | Definition | Frontend Application |
|-----------|-----------|----------------------|
| **SRP** — Single Responsibility | A module/component should have one reason to change | One component = one UI concern. Separate data fetching from rendering. |
| **OCP** — Open/Closed | Open for extension, closed for modification | Use composition, render props, slots, and higher-order components instead of modifying shared components. |
| **LSP** — Liskov Substitution | Subtypes must be substitutable for their base types | A `<PrimaryButton>` should be usable anywhere a `<Button>` is expected without breaking the interface contract. |
| **ISP** — Interface Segregation | No client should depend on methods it doesn't use | Keep component prop interfaces minimal. Avoid "god props" objects. Split large context providers. |
| **DIP** — Dependency Inversion | Depend on abstractions, not concretions | Components depend on service interfaces (hooks, context), not on `fetch()` or `localStorage` directly. |

### 1.2 Frontend-Specific Principles

#### Separation of Concerns (SoC)

```
┌─────────────────────────────────────────────┐
│                  UI Layer                    │
│  Components · Styles · Animations · Layout  │
├─────────────────────────────────────────────┤
│              State Layer                     │
│  Local State · Global Store · Cache · URL   │
├─────────────────────────────────────────────┤
│              Service Layer                   │
│  API Clients · Auth · Validation · i18n     │
├─────────────────────────────────────────────┤
│            Infrastructure Layer              │
│  Routing · Build · Env Config · Analytics   │
└─────────────────────────────────────────────┘
```

- **Structure**: Markup/templates handle document structure.
- **Presentation**: CSS/styling handles visual appearance.
- **Behavior**: JavaScript handles interactivity and logic.
- **Data**: State management handles application data flow.

#### Composition Over Inheritance

- Prefer composing small components over extending base classes.
- Use hooks (React), composables (Vue 3), or services (Angular) for shared logic.
- Build complex UI from simple, composable primitives.

#### Unidirectional Data Flow

```
Action → Dispatcher → Store → View → Action
```

- Data flows in one direction, making state changes predictable and traceable.
- Parent-to-child data passing via props; child-to-parent communication via events/callbacks.
- Eliminates two-way binding complexity in large applications.

#### Principle of Least Privilege (UI Security)

- Components receive only the data and permissions they need.
- Never trust client-side validation alone — always validate on the server.
- Sanitize user-generated content to prevent XSS.
- Avoid exposing sensitive data in client bundles, URLs, or local storage.

#### Colocation

- Place related code close together: component, styles, tests, types, stories in the same directory.
- Reduces cognitive overhead and makes refactoring safer.

#### Progressive Enhancement & Graceful Degradation

- **Progressive Enhancement**: Start with a baseline HTML experience, layer on CSS and JS.
- **Graceful Degradation**: Build the full experience first, ensure it doesn't break in constrained environments.
- Both strategies ensure maximum accessibility and resilience.

### 1.3 Functional Programming Principles in the Frontend

| Concept | Application |
|---------|-------------|
| **Pure Functions** | Reducers, selectors, utility functions — same input always produces same output. |
| **Immutability** | Never mutate state directly. Use spread operators, `structuredClone`, or libraries like Immer. |
| **Declarative UI** | Describe *what* the UI should look like, not *how* to manipulate the DOM. |
| **Higher-Order Functions** | HOCs, custom hooks, middleware, pipe/compose utilities. |
| **Referential Transparency** | Components with identical props render identically — enables memoization. |

### 1.4 Resilience Principles

- **Fail gracefully**: Use error boundaries and fallback UI.
- **Retry with backoff**: Transient network failures should not crash the app.
- **Timeout management**: Every external call should have a timeout.
- **Circuit breaker (client-side)**: Stop hammering a failing service; surface degraded state.
- **Offline-first mindset**: Leverage service workers, IndexedDB, and optimistic UI.

---

## 2. Pattern Catalog

### 2.1 Component Patterns

#### Presentational vs. Container Components

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

#### Compound Components

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

#### Render Props / Scoped Slots

Delegate rendering control to the consumer while the component manages logic.

```jsx
<DataFetcher url="/api/users">
  {({ data, loading, error }) => (
    loading ? <Spinner /> : <UserList users={data} />
  )}
</DataFetcher>
```

#### Higher-Order Components (HOC)

A function that takes a component and returns an enhanced component.

```
withAuth(Component)  → Adds authentication gate
withTheme(Component) → Injects theme context
withLogger(Component) → Adds lifecycle logging
```

**Modern alternative**: Custom hooks / composables are now preferred over HOCs.

#### Headless Components (Renderless)

Components that encapsulate behavior with zero UI opinions. The consumer provides all rendering.

```
useCombobox()   → Manages keyboard nav, filtering, selection
useDialog()     → Manages open/close, focus trap, escape key
usePagination() → Manages page state, boundaries, navigation
```

Libraries: Headless UI, Radix, Downshift, TanStack Table.

#### Provider Pattern

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

#### Controlled vs. Uncontrolled Components

| Aspect | Controlled | Uncontrolled |
|--------|-----------|-------------|
| State owner | Parent component | DOM / internal ref |
| Data flow | Props + onChange | Ref-based access |
| Validation | Real-time | On submit |
| Use case | Forms with complex logic | Simple forms, file inputs |

### 2.2 State Management Patterns

#### Local Component State

- `useState`, `useReducer` (React), `ref()`, `reactive()` (Vue), class properties (Angular).
- Default choice — lift state only when necessary.

#### Lifted State / Prop Drilling

- Move shared state to the nearest common ancestor.
- Acceptable for <3 levels of depth. Beyond that, use context or state management.

#### Global Store (Flux / Redux Pattern)

```
┌────────┐  dispatch  ┌────────────┐  reduce  ┌───────┐  select  ┌──────┐
│ Action │──────────▶ │ Dispatcher │────────▶ │ Store │────────▶ │ View │
└────────┘            └────────────┘          └───────┘          └──────┘
     ▲                                                               │
     └───────────────────────────────────────────────────────────────┘
```

Libraries: Redux (Toolkit), Zustand, Pinia, NgRx, MobX.

#### Atomic State

State is broken into independent atoms that can be composed.

```
const userAtom = atom(null)
const themeAtom = atom('light')
const derivedAtom = computed(() => ...)  // derived state
```

Libraries: Jotai, Recoil, Nanostores.

#### Server State (Cache-First)

Treats remote data as a cache, not as application state.

```
┌─────────┐  fetch/mutate  ┌──────────┐  hydrate  ┌──────┐
│  Server │◀──────────────▶│  Cache   │─────────▶ │  UI  │
└─────────┘                └──────────┘           └──────┘
                           (stale-while-revalidate)
```

Libraries: TanStack Query, SWR, RTK Query, Apollo Client.

Key concepts: caching, background refetching, optimistic updates, pagination, infinite scroll.

#### State Machines / Statecharts

Model complex UI flows as finite state machines.

```
idle → loading → success
                → error → retry → loading
```

Eliminates impossible states. Libraries: XState, Robot.

#### URL as State

- The URL is the single source of truth for navigation-related state.
- Search params, hash fragments, and path segments encode application state.
- Enables deep linking, sharing, and back/forward navigation.

### 2.3 Data Fetching Patterns

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Fetch-on-render** | Component mounts, then fetches | Simple cases, client-only apps |
| **Fetch-then-render** | Fetch completes before route renders | SSR, loaders (Remix, Next.js) |
| **Render-as-you-fetch** | Fetch starts early, component renders in parallel | Suspense-based architectures |
| **Prefetching** | Fetch on hover/focus before navigation | Predictive UX optimization |
| **Optimistic updates** | UI updates instantly, rolls back on failure | Mutations where latency matters |
| **Polling** | Repeated fetches at interval | Dashboards, status pages |
| **WebSocket / SSE** | Server pushes updates to client | Real-time: chat, notifications, collaboration |

### 2.4 Rendering Patterns

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

### 2.5 Cross-Cutting Patterns

#### Error Handling Strategy

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

#### Authentication / Authorization Patterns

- **Route guards**: Protect routes based on auth state.
- **Permission-based rendering**: Show/hide UI elements based on roles.
- **Token refresh**: Silent refresh with interceptors.
- **Session management**: Secure cookie-based or token-based flows.

#### Internationalization (i18n)

- Extract strings into locale files (JSON, ICU format).
- Handle pluralization, date/number formatting, RTL layouts.
- Lazy-load locale bundles to minimize payload.

#### Feature Flags

- Decouple deployment from release.
- Gradual rollouts, A/B testing, kill switches.
- Libraries: LaunchDarkly, Unleash, Flagsmith; or a simple JSON config with a context provider.

---

## 3. Architecture Styles

### 3.1 Monolithic SPA

```
┌─────────────────────────────────────────┐
│              Single Bundle               │
│  ┌─────┐ ┌──────┐ ┌────────┐ ┌──────┐  │
│  │Auth │ │Users │ │Products│ │Orders│  │
│  └─────┘ └──────┘ └────────┘ └──────┘  │
│          Shared Router & Store          │
└─────────────────────────────────────────┘
```

| Pros | Cons |
|------|------|
| Simple mental model | Large bundle size at scale |
| Easy to develop early on | Slow CI/CD as codebase grows |
| Shared dependencies | Team coupling — one repo, one deploy |
| Straightforward testing | Difficult to incrementally adopt new tech |

**Best for**: Small-to-medium apps, single team, <50K LOC.

### 3.2 Modular Monolith (Feature-Sliced)

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── store/
│   │   ├── types/
│   │   └── index.ts         ← public API
│   ├── dashboard/
│   ├── settings/
│   └── ...
├── shared/
│   ├── ui/                   ← design system primitives
│   ├── lib/                  ← utilities
│   └── api/                  ← HTTP client
├── app/
│   ├── router.ts
│   ├── providers.ts
│   └── App.tsx
└── index.ts
```

**Rules**:
1. Features never import from other features directly — communicate through shared contracts or events.
2. `shared/` contains only truly universal code.
3. Each feature's `index.ts` is its public API — internal modules are not importable.

**Enforcement**: ESLint boundaries plugin, path aliases, barrel exports.

### 3.3 Micro-Frontends

```
┌─────────────────────────────────────────────────────┐
│                  App Shell / Orchestrator            │
├────────────┬────────────┬────────────┬──────────────┤
│  Team A    │  Team B    │  Team C    │  Team D      │
│  React 18  │  Vue 3     │  React 19  │  Svelte      │
│  Auth MFE  │  Catalog   │  Checkout  │  Analytics   │
│  Deploy ⚡  │  Deploy ⚡  │  Deploy ⚡  │  Deploy ⚡    │
└────────────┴────────────┴────────────┴──────────────┘
```

#### Composition Strategies

| Strategy | Integration Point | Example |
|----------|-------------------|---------|
| **Build-time** | npm packages | Shared component library versioned as packages |
| **Server-side** | HTML stitching | SSI, ESI, Tailor, Podium |
| **Run-time (iframe)** | Isolated frames | Legacy integration; total isolation |
| **Run-time (JS)** | Module Federation / Import Maps | Webpack Module Federation, Native Federation |
| **Edge-side** | CDN composition | Edge-side includes, Cloudflare Workers |

#### Communication Between MFEs

- **Custom Events**: Loosely coupled, pub/sub via `CustomEvent`.
- **Shared State Bus**: Lightweight event emitter or observable.
- **URL / Query Params**: For navigation-driven state sharing.
- **PostMessage**: For iframe-based isolation.

**Avoid**: Shared global stores — they create hidden coupling.

**Best for**: Large organizations, multiple autonomous teams, heterogeneous tech stacks.

### 3.4 Component-Driven Architecture

```
┌──────────────────────────────────────┐
│           Application Pages          │
├──────────────────────────────────────┤
│         Feature Components           │
│    (domain-specific composites)      │
├──────────────────────────────────────┤
│         Pattern Components           │
│    (cards, forms, modals, tables)    │
├──────────────────────────────────────┤
│         Primitive Components         │
│  (buttons, inputs, icons, badges)    │
├──────────────────────────────────────┤
│           Design Tokens              │
│  (colors, spacing, typography)       │
└──────────────────────────────────────┘
```

- Build from small, reusable atoms up to pages (Atomic Design).
- Use a component catalog (Storybook) as the living documentation.
- Enforce visual consistency through design tokens and a shared design system.

### 3.5 Jamstack / Edge Architecture

```
┌─────────────┐    ┌──────┐    ┌───────────┐
│   CDN Edge  │◀───│ Build│◀───│   Git Repo│
│  (static +  │    └──────┘    └───────────┘
│   functions)│
│             │◀──▶ APIs / BaaS / Headless CMS
└─────────────┘
```

- Pre-render where possible, compute at the edge, call APIs for dynamic data.
- Frameworks: Next.js, Nuxt, Astro, SvelteKit, Remix.
- Benefits: Performance, security (smaller attack surface), scalability, developer experience.

### 3.6 Hybrid / Transitional Architecture

Modern meta-frameworks blur boundaries:

| Capability | Next.js App Router | Nuxt 3 | SvelteKit | Remix |
|------------|-------------------|--------|-----------|-------|
| SSG | ✅ | ✅ | ✅ | ✅ |
| SSR | ✅ | ✅ | ✅ | ✅ |
| ISR | ✅ | ✅ (route rules) | ✅ | — |
| Streaming | ✅ | ✅ | ✅ | ✅ |
| Server Actions | ✅ | ✅ (server utils) | ✅ (form actions) | ✅ (actions) |
| Edge Runtime | ✅ | ✅ | ✅ | ✅ |
| File-system routing | ✅ | ✅ | ✅ | ✅ |

**Key insight**: Choose rendering strategy **per route**, not per application.

---

## 4. Performance Architecture

### 4.1 Core Web Vitals — The North Star Metrics

| Metric | What it Measures | Target | Primary Lever |
|--------|-----------------|--------|---------------|
| **LCP** (Largest Contentful Paint) | Loading speed | ≤ 2.5s | Optimize critical resource delivery |
| **INP** (Interaction to Next Paint) | Responsiveness | ≤ 200ms | Reduce JS execution on main thread |
| **CLS** (Cumulative Layout Shift) | Visual stability | ≤ 0.1 | Reserve space for dynamic content |

### 4.2 Bundle Architecture

```
                    ┌──────────────┐
                    │  Entry Point │
                    └──────┬───────┘
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
     ┌────────────┐ ┌───────────┐ ┌────────────┐
     │  Vendor    │ │  Runtime  │ │  App Core  │
     │  (cached)  │ │  (small)  │ │  (critical)│
     └────────────┘ └───────────┘ └────────────┘
                                        │
                        ┌───────────────┼───────────────┐
                        ▼               ▼               ▼
                  ┌──────────┐   ┌──────────┐   ┌──────────┐
                  │ Route A  │   │ Route B  │   │ Route C  │
                  │ (lazy)   │   │ (lazy)   │   │ (lazy)   │
                  └──────────┘   └──────────┘   └──────────┘
```

**Strategies**:
- **Code splitting**: Route-based, component-based (`lazy()`/`defineAsyncComponent`).
- **Tree shaking**: Dead code elimination via ES modules. Avoid side-effects in modules.
- **Bundle analysis**: Use `webpack-bundle-analyzer`, `source-map-explorer`, or `vite-plugin-visualizer`.
- **Dynamic imports**: Load heavy libraries (charts, editors, maps) on demand.
- **Module/nomodule**: Serve modern bundles to modern browsers, legacy to old.

### 4.3 Network Performance

| Technique | Description |
|-----------|-------------|
| **CDN** | Serve static assets from edge locations nearest to user |
| **Compression** | Brotli (preferred) or Gzip for all text assets |
| **HTTP/2 & HTTP/3** | Multiplexing eliminates request queuing |
| **Preload / Prefetch** | `<link rel="preload">` for critical assets, `prefetch` for next-page assets |
| **Resource hints** | `preconnect`, `dns-prefetch` for third-party origins |
| **Service Workers** | Cache-first strategies for returning users; offline support |
| **Stale-While-Revalidate** | Serve from cache, revalidate in background |

### 4.4 Rendering Performance

- **Virtual DOM diffing**: Understand reconciliation — key props, avoiding unnecessary re-renders.
- **Memoization**: `React.memo`, `useMemo`, `useCallback`, `computed()` — use judiciously, not by default.
- **Virtualization**: Render only visible items in large lists/tables. Libraries: TanStack Virtual, react-window.
- **Debounce & Throttle**: Control rate of expensive operations (scroll, resize, input handlers).
- **Web Workers**: Off-load CPU-intensive tasks (parsing, compression, image processing).
- **requestAnimationFrame**: Synchronize visual updates with browser paint cycle.
- **CSS containment**: `contain: layout style paint` to isolate rendering subtrees.
- **Content-visibility**: `content-visibility: auto` to skip rendering offscreen content.

### 4.5 Asset Optimization

| Asset | Optimization Strategy |
|-------|----------------------|
| **Images** | Modern formats (WebP, AVIF), responsive `srcset`, lazy loading, CDN image transforms |
| **Fonts** | `font-display: swap`, subset only needed glyphs, preload critical fonts, self-host |
| **CSS** | Critical CSS inlined, remaining async-loaded, PurgeCSS for unused styles |
| **JavaScript** | Minification, scope hoisting, sideEffects flag, differential serving |
| **SVG** | Inline for icons, optimize with SVGO, use sprite sheets for many icons |

### 4.6 Performance Monitoring

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Lab Data    │    │  Field Data  │    │  Alerting    │
│  (Synthetic) │    │  (RUM)       │    │              │
│              │    │              │    │              │
│  Lighthouse  │    │  CrUX        │    │  Budgets CI  │
│  WebPageTest │    │  Sentry Perf │    │  Regression  │
│  DevTools    │    │  Datadog RUM │    │  Slack/PD    │
└──────────────┘    └──────────────┘    └──────────────┘
```

**Performance budgets**: Set and enforce in CI.

```
Metric           │ Budget
─────────────────┼────────
JS (compressed)  │ < 200 KB
CSS (compressed) │ < 50 KB
LCP              │ < 2.5s
INP              │ < 200ms
CLS              │ < 0.1
TTI              │ < 3.5s
```

---

## 5. Team Standards

### 5.1 Code Quality Toolchain

```
┌──────────────────────────────────────────────────┐
│                    Developer Workflow             │
│                                                  │
│  Editor          Pre-Commit       CI Pipeline    │
│  ┌───────────┐   ┌───────────┐   ┌───────────┐  │
│  │ ESLint    │   │ lint-staged│   │ Lint      │  │
│  │ Prettier  │   │ Husky     │   │ Type Check│  │
│  │ TypeScript│   │ commitlint│   │ Tests     │  │
│  │ Stylelint │   │           │   │ Build     │  │
│  │ EditorCfg │   │           │   │ Audit     │  │
│  └───────────┘   └───────────┘   └───────────┘  │
└──────────────────────────────────────────────────┘
```

### 5.2 TypeScript Standards

| Practice | Guidance |
|----------|----------|
| **Strict mode** | Always enable `strict: true` in `tsconfig.json` |
| **Explicit return types** | On public API functions and exported hooks |
| **Discriminated unions** | Model states with tagged unions, not boolean flags |
| **`unknown` over `any`** | Force type narrowing; ban `any` via ESLint rule |
| **Branded types** | Distinguish `UserId` from `OrderId` at the type level |
| **Generics** | Use for reusable utilities; avoid overuse that hurts readability |
| **`satisfies` operator** | Validate types at assignment without widening |
| **Zod / Valibot** | Runtime validation at system boundaries (API responses) |

### 5.3 Testing Strategy — The Testing Trophy

```
            ┌──────────────┐
            │   E2E Tests  │   Few — critical user journeys
            ├──────────────┤
         ┌──┤ Integration  ├──┐   Many — test components with
         │  │   Tests      │  │   their collaborators
         │  ├──────────────┤  │
         │  │  Unit Tests  │  │   Focused — pure logic, utils,
         │  │              │  │   reducers, hooks
         │  ├──────────────┤  │
         └──┤ Static       ├──┘   Foundation — TypeScript, ESLint
            │ Analysis     │
            └──────────────┘
```

| Layer | Scope | Tools | Speed |
|-------|-------|-------|-------|
| **Static** | Type errors, lint rules | TypeScript, ESLint | Instant |
| **Unit** | Pure functions, utilities | Vitest, Jest | Fast |
| **Integration** | Component + hooks + state | Testing Library, MSW | Medium |
| **E2E** | Full user flows | Playwright, Cypress | Slow |
| **Visual Regression** | Pixel-level UI changes | Chromatic, Percy, Playwright screenshots | Medium |

**Key principle**: *Write tests that resemble how users interact with your software.* Avoid testing implementation details.

### 5.4 CSS / Styling Architecture

| Approach | Scoping | Runtime Cost | DX | Best For |
|----------|---------|-------------|-----|----------|
| **CSS Modules** | File-scoped | None | Good | Medium apps, Vite/Webpack |
| **Tailwind CSS** | Utility classes | None | Excellent | Rapid prototyping, design systems |
| **CSS-in-JS (zero-runtime)** | File-scoped | None | Good | Vanilla Extract, Panda CSS |
| **CSS-in-JS (runtime)** | Component-scoped | Yes | Great | styled-components, Emotion (use sparingly) |
| **BEM** | Convention-scoped | None | Moderate | Legacy projects, vanilla CSS |
| **Scoped styles** | Framework-scoped | None | Great | Vue `<style scoped>`, Angular ViewEncapsulation |

**Design Tokens** — The single source of truth for visual design:
```css
:root {
  --color-primary: #0066ff;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --radius-md: 8px;
  --font-body: 'Inter', sans-serif;
  --shadow-md: 0 4px 6px rgb(0 0 0 / 0.1);
}
```

### 5.5 Git & Branching Strategy

**Commit Conventions** (Conventional Commits):
```
<type>(<scope>): <description>

feat(auth): add OAuth2 PKCE login flow
fix(cart): correct quantity calculation on update
perf(images): add lazy loading to product gallery
chore(deps): upgrade React to v19
refactor(api): extract HTTP client into service layer
```

**Branching Models**:
- **Trunk-based development** (preferred): Short-lived feature branches, merge to `main` daily, feature flags for incomplete work.
- **Git Flow**: For teams with rigid release cycles — `develop`, `release/*`, `hotfix/*`.

### 5.6 Code Review Standards

| Area | What to Check |
|------|---------------|
| **Correctness** | Does it do what it claims? Edge cases handled? |
| **Security** | XSS vectors, auth checks, sensitive data exposure, dependency risks |
| **Performance** | Unnecessary re-renders, large bundles, unoptimized assets |
| **Accessibility** | Semantic HTML, ARIA, keyboard navigation, color contrast |
| **Readability** | Clear naming, reasonable abstractions, not over-engineered |
| **Tests** | Meaningful coverage for new logic; no snapshot-only tests |
| **Types** | No `any` escapes, proper null handling, discriminated unions |

### 5.7 Accessibility (a11y) Standards

**WCAG 2.2 Compliance Levels**: A (minimum) → AA (standard target) → AAA (enhanced).

| Category | Requirements |
|----------|-------------|
| **Semantic HTML** | Use `<button>`, `<nav>`, `<main>`, `<article>` — not `<div>` for everything |
| **Keyboard navigation** | All interactive elements reachable and operable via keyboard |
| **Focus management** | Visible focus indicators, logical focus order, focus trapping in modals |
| **ARIA** | Use ARIA when HTML semantics are insufficient. Don't over-ARIA. |
| **Color contrast** | 4.5:1 for normal text, 3:1 for large text (AA) |
| **Screen readers** | Test with VoiceOver, NVDA, JAWS |
| **Motion** | Respect `prefers-reduced-motion` |
| **Forms** | Visible labels, error messages associated with inputs, required fields indicated |

**Tooling**: axe-core in CI, Lighthouse a11y audit, eslint-plugin-jsx-a11y, Pa11y.

### 5.8 Documentation Standards

| Artifact | Tool / Format | Purpose |
|----------|---------------|---------|
| **Component catalog** | Storybook, Histoire, Ladle | Interactive showcase of all UI components |
| **API docs** | TypeDoc, TSDoc comments | Auto-generated from TypeScript types |
| **ADRs** | Markdown in `docs/adr/` | Record architectural decisions and rationale |
| **Runbooks** | Markdown / Wiki | How to deploy, roll back, debug production |
| **README** | Markdown | Setup, architecture overview, conventions |
| **Changelog** | Auto-generated | Track user-facing changes per release |

### 5.9 Monorepo Architecture

```
my-org/
├── apps/
│   ├── web/                  ← main SPA
│   ├── admin/                ← admin dashboard
│   └── mobile/               ← React Native
├── packages/
│   ├── ui/                   ← shared design system
│   ├── api-client/           ← typed API client
│   ├── config-eslint/        ← shared ESLint config
│   ├── config-ts/            ← shared tsconfig
│   └── utils/                ← shared utilities
├── turbo.json / nx.json      ← task orchestration
├── pnpm-workspace.yaml       ← workspace definition
└── package.json
```

**Tooling**: Turborepo, Nx, Lerna, pnpm workspaces.

**Benefits**: Code sharing, atomic changes across packages, unified CI, consistent tooling.

**Boundaries**: Enforce package dependencies via `depcheck`, `eslint-plugin-boundaries`, or Nx module boundaries.

---

## 6. Suggested Learning Roadmap by Career Level

### Level 1 — Junior Frontend Developer (0–2 years)

**Goal**: Build functional, correct, accessible UIs with confidence.

```
Phase 1: Foundations
├── HTML semantics & accessibility fundamentals
├── CSS layout (Flexbox, Grid), responsive design, media queries
├── JavaScript fundamentals (ES2015+), DOM API
├── TypeScript basics (types, interfaces, generics)
└── Browser DevTools proficiency

Phase 2: Framework Mastery
├── Core framework (React / Vue / Angular) — pick one deeply
├── Component lifecycle, state, props, events
├── Routing (client-side navigation)
├── Form handling & validation
└── Basic API integration (fetch, async/await)

Phase 3: Professional Practices
├── Git workflows & conventional commits
├── Testing fundamentals (unit + integration with Testing Library)
├── CSS methodology (Modules, Tailwind, or BEM)
├── Package managers (npm, pnpm) & basic build tool usage
└── Code review participation (giving & receiving feedback)
```

**Milestone**: Can independently build and ship a feature end-to-end within an existing codebase.

---

### Level 2 — Mid-Level Frontend Developer (2–5 years)

**Goal**: Design robust features, own subsystems, and mentor juniors.

```
Phase 1: Deeper Patterns
├── Advanced state management (global store, server state, state machines)
├── Design patterns (compound components, render props, headless)
├── Error handling strategy (error boundaries, retry logic)
├── Authentication / authorization patterns
└── SRP & composition over inheritance in component design

Phase 2: Performance & Quality
├── Core Web Vitals — understanding & optimizing
├── Bundle optimization (code splitting, lazy loading, tree shaking)
├── Rendering performance (memoization, virtualization, profiling)
├── E2E testing (Playwright / Cypress)
├── Visual regression testing
└── Accessibility auditing & remediation

Phase 3: Tooling & Architecture
├── Build tools deep dive (Vite, Webpack, esbuild)
├── Monorepo basics (workspaces, shared packages)
├── CI/CD pipeline configuration
├── Feature-sliced project structure
├── API design awareness (REST, GraphQL, tRPC)
└── Observability (error tracking, logging, performance monitoring)
```

**Milestone**: Can design and implement a complex feature (e.g., multi-step form, real-time dashboard) with proper architecture, tests, performance, and accessibility.

---

### Level 3 — Senior Frontend Developer (5–8 years)

**Goal**: Architect features and subsystems; set technical direction for features; elevate team quality.

```
Phase 1: Architecture & Design
├── Modular monolith / feature-sliced architecture
├── Rendering strategy selection (CSR vs SSR vs SSG vs ISR per route)
├── Design system architecture (tokens, primitives, patterns, documentation)
├── State architecture for complex domains
├── API layer design (client generation, caching strategy, error contracts)
└── Security patterns (CSP, XSS prevention, auth flows, dependency auditing)

Phase 2: System Thinking
├── Performance budgets & regression prevention in CI
├── Bundle architecture & dependency management at scale
├── Third-party script management & vendor isolation
├── Cross-cutting concerns (i18n, feature flags, analytics, A/B testing)
├── Migration strategies (incremental adoption, strangler fig)
└── Technical debt identification, prioritization, and communication

Phase 3: Leadership
├── Architectural Decision Records (ADRs)
├── Mentoring junior and mid-level developers
├── Code review culture & standards
├── RFC process for significant changes
├── Cross-team collaboration & alignment
└── Hiring: technical interviews and assessment
```

**Milestone**: Can own the architecture of a large feature area, write RFCs, establish patterns adopted by the team, and independently drive technical decisions.

---

### Level 4 — Staff / Principal Frontend Engineer (8+ years)

**Goal**: Define organizational frontend strategy; solve cross-cutting technical challenges; influence beyond a single team.

```
Phase 1: Organizational Architecture
├── Micro-frontend architecture design & governance
├── Platform/infrastructure team creation (DX tooling, shared libraries)
├── Cross-team design system strategy (multi-brand, multi-platform)
├── Build & deploy infrastructure at scale (CI optimization, caching, preview envs)
├── Monorepo governance (dependency policies, package boundaries, versioning)
└── Framework & library evaluation process (build vs. buy analysis)

Phase 2: Strategic Thinking
├── Multi-year technical roadmapping
├── Technology radar & ecosystem monitoring
├── Performance as an organization-wide practice (RUM dashboards, OKRs)
├── Cross-platform strategy (web, mobile, desktop, embedded)
├── Legacy modernization strategies at organizational scale
└── Developer experience (DX) as a force multiplier

Phase 3: Influence & Impact
├── Engineering blog posts, conference talks, OSS contributions
├── Defining engineering levels, competency matrices, and growth frameworks
├── Partnering with Product, Design, and Backend on system-level decisions
├── Vendor and partner technical evaluations
├── Incident response & post-mortem culture
└── Building a culture of quality, experimentation, and continuous improvement
```

**Milestone**: Your architectural decisions and standards are adopted across multiple teams. You are the go-to person for "How should we build this?" You leave systems better than you found them and grow the engineers around you.

---

## Quick Reference: Decision Matrix

### When to Use What Rendering Strategy

```
Is the content public & SEO-critical?
├── Yes → Does it change rarely?
│         ├── Yes → SSG / ISR
│         └── No  → SSR (streaming)
└── No  → Is it a highly interactive app?
          ├── Yes → CSR (SPA) with code splitting
          └── No  → SSR or hybrid
```

### When to Use What State Management

```
Is the state local to one component?
├── Yes → useState / ref()
└── No  → Is it shared across a few siblings?
          ├── Yes → Lift state to parent
          └── No  → Is it server/remote data?
                    ├── Yes → TanStack Query / SWR
                    └── No  → Is it global app state?
                              ├── Yes → Zustand / Pinia / Redux Toolkit
                              └── No  → URL state / context
```

### When to Adopt Micro-Frontends

```
Do you have 3+ autonomous teams working on the same product?
├── No  → Use a modular monolith
└── Yes → Do teams need independent deploy cycles?
          ├── No  → Modular monolith with clear boundaries
          └── Yes → Do teams need technology autonomy?
                    ├── No  → Monorepo with independent CI pipelines
                    └── Yes → Micro-frontends
```

---

> **This framework is a living document.** Revisit and update it as the frontend ecosystem evolves. The best architecture is the one that serves your team, your users, and your constraints — not the one with the most buzzwords.
