# 1. Principles Framework

[← Back to Index](frontend-engineering-framework.md)

---

## 1.1 Core Software Design Principles

| Principle | Definition | Frontend Application |
|-----------|-----------|----------------------|
| **SRP** — Single Responsibility | A module/component should have one reason to change | One component = one UI concern. Separate data fetching from rendering. |
| **OCP** — Open/Closed | Open for extension, closed for modification | Use composition, render props, slots, and higher-order components instead of modifying shared components. |
| **LSP** — Liskov Substitution | Subtypes must be substitutable for their base types | A `<PrimaryButton>` should be usable anywhere a `<Button>` is expected without breaking the interface contract. |
| **ISP** — Interface Segregation | No client should depend on methods it doesn't use | Keep component prop interfaces minimal. Avoid "god props" objects. Split large context providers. |
| **DIP** — Dependency Inversion | Depend on abstractions, not concretions | Components depend on service interfaces (hooks, context), not on `fetch()` or `localStorage` directly. |

---

## 1.2 General Engineering Principles

### DRY — Don't Repeat Yourself

> "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."

- Extract shared logic into custom hooks, utilities, or shared components.
- Centralize constants, type definitions, and configuration.
- **Frontend caveat**: Don't over-DRY presentational code. Two components that look the same today may diverge tomorrow. Duplicate first, abstract later — only when the pattern is proven.

### WET — Write Everything Twice

> "Duplication is far cheaper than the wrong abstraction."

- Allow up to 2–3 duplications before extracting a shared abstraction.
- Premature abstraction creates rigid, hard-to-change code.
- Use WET as a **threshold signal**: the third duplication is when you abstract.

### KISS — Keep It Simple, Stupid

- Choose the simplest solution that satisfies the requirement.
- Avoid clever one-liners that sacrifice readability.
- Prefer explicit code over implicit magic (e.g., explicit props over deep injection).
- A junior developer should understand your component within a few minutes.

### YAGNI — You Aren't Gonna Need It

- Don't build features, abstractions, or configurations speculatively.
- Avoid adding optional parameters "just in case" — extend when the need is real.
- Applies to: premature generalization, unused component variants, over-configurable props, speculative API fields.

### LoD — Law of Demeter (Principle of Least Knowledge)

> "Only talk to your immediate friends."

- A component should not reach deep into nested objects: avoid `user.address.city.zipCode` chains.
- Pass only what the component needs — not entire parent objects.
- Use selectors or derived state to flatten data before passing to UI.

```jsx
// ❌ Violates LoD — component knows too much about data shape
<OrderCard total={order.payment.summary.total} />

// ✅ Respects LoD — parent extracts what's needed
const total = selectOrderTotal(order)
<OrderCard total={total} />
```

### CQS — Command-Query Separation

- **Queries** return data and have no side effects (getters, selectors, computed values).
- **Commands** perform actions and change state (event handlers, mutations, dispatches).
- Never mix both: a function should either return something or change something, not both.

```ts
// ❌ Mixed — returns value AND mutates
function addItemAndGetTotal(item: Item): number {
  cart.push(item)        // command
  return cart.totalPrice // query
}

// ✅ Separated
function addItem(item: Item): void { cart.push(item) }
function getTotal(): number { return cart.totalPrice }
```

### Separation of Concerns (Broad Principle)

- Each module, layer, or component addresses a distinct concern.
- In frontend: separate **data fetching** from **presentation**, **business logic** from **UI logic**, **styling** from **structure**.
- Framework-specific: React hooks separate logic from JSX; Vue composables separate logic from templates; Angular services separate logic from components.

### Encapsulation

- Hide internal implementation details; expose only a public API.
- Components expose props/events — not internal state or refs.
- CSS Modules/scoped styles encapsulate visual details.
- Barrel exports (`index.ts`) define the public surface of a feature module.

```
feature/
├── components/       ← internal
├── hooks/            ← internal
├── utils/            ← internal
└── index.ts          ← public API (only this is importable)
```

### Principle of Least Astonishment (POLA)

- Code should behave the way other developers expect.
- Name functions and components clearly — `useAuth()` should relate to authentication, not analytics.
- Follow framework conventions: a React hook starts with `use`, a Vue composable starts with `use`, an Angular service is injectable.
- Default behavior should be safe and unsurprising.

### Robustness Principle (Postel's Law)

> "Be conservative in what you send, be liberal in what you accept."

- Components should accept flexible input (optional props, sensible defaults) but produce strict, predictable output.
- API clients should handle unexpected response shapes gracefully (validate at the boundary with Zod/Valibot).
- Useful for building resilient shared libraries and design system components.

### Inversion of Control (IoC)

- The framework calls your code, not the other way around.
- Frontend examples: React renders your components, the router calls your loaders, middleware processes your actions.
- **Hooks / composables** are IoC — the framework orchestrates their lifecycle.
- **Dependency injection** (Angular) and **context** (React/Vue) are IoC mechanisms for decoupling consumers from implementations.

### AHA — Avoid Hasty Abstractions

> "Prefer duplication over the wrong abstraction." — Sandi Metz

- Wait until you see the actual shape of variation before abstracting.
- Optimize for change, not for reducing lines of code.
- Wrong abstractions are more expensive to fix than duplication.

### Principle Tension Map

Principles often compete. Use judgment based on context:

```
DRY ◄──────────────────────────────► WET / AHA
"Remove all duplication"             "Duplicate until the pattern is clear"

YAGNI ◄────────────────────────────► Extensibility (OCP)
"Don't build it yet"                 "Design for future extension"

KISS ◄─────────────────────────────► Robustness
"Keep it simple"                     "Handle all edge cases"

Encapsulation ◄────────────────────► Transparency
"Hide internals"                     "Make behavior observable/debuggable"
```

**Rule of thumb**: Start simple (KISS, YAGNI, WET). Add complexity only when proven necessary (DRY, OCP, robustness). The best code is the minimum code that correctly solves the current problem.

---

## 1.3 Frontend-Specific Principles

### Separation of Concerns (SoC) — Applied

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

### Composition Over Inheritance

- Prefer composing small components over extending base classes.
- Use hooks (React), composables (Vue 3), or services (Angular) for shared logic.
- Build complex UI from simple, composable primitives.

### Unidirectional Data Flow

```
Action → Dispatcher → Store → View → Action
```

- Data flows in one direction, making state changes predictable and traceable.
- Parent-to-child data passing via props; child-to-parent communication via events/callbacks.
- Eliminates two-way binding complexity in large applications.

### Principle of Least Privilege (UI Security)

- Components receive only the data and permissions they need.
- Never trust client-side validation alone — always validate on the server.
- Sanitize user-generated content to prevent XSS.
- Avoid exposing sensitive data in client bundles, URLs, or local storage.

### Colocation

- Place related code close together: component, styles, tests, types, stories in the same directory.
- Reduces cognitive overhead and makes refactoring safer.

### Progressive Enhancement & Graceful Degradation

- **Progressive Enhancement**: Start with a baseline HTML experience, layer on CSS and JS.
- **Graceful Degradation**: Build the full experience first, ensure it doesn't break in constrained environments.
- Both strategies ensure maximum accessibility and resilience.

---

## 1.4 Functional Programming Principles in the Frontend

| Concept | Application |
|---------|-------------|
| **Pure Functions** | Reducers, selectors, utility functions — same input always produces same output. |
| **Immutability** | Never mutate state directly. Use spread operators, `structuredClone`, or libraries like Immer. |
| **Declarative UI** | Describe *what* the UI should look like, not *how* to manipulate the DOM. |
| **Higher-Order Functions** | HOCs, custom hooks, middleware, pipe/compose utilities. |
| **Referential Transparency** | Components with identical props render identically — enables memoization. |

---

## 1.5 Resilience Principles

- **Fail gracefully**: Use error boundaries and fallback UI.
- **Retry with backoff**: Transient network failures should not crash the app.
- **Timeout management**: Every external call should have a timeout.
- **Circuit breaker (client-side)**: Stop hammering a failing service; surface degraded state.
- **Offline-first mindset**: Leverage service workers, IndexedDB, and optimistic UI.

---

Next: [Pattern Catalog →](02-pattern-catalog.md)
