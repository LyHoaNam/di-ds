# 3. Architecture Styles

[← Back to Index](frontend-engineering-framework.md)

---

## 3.1 Monolithic SPA

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

---

## 3.2 Modular Monolith (Feature-Sliced)

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

---

## 3.3 Micro-Frontends

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

### Composition Strategies

| Strategy | Integration Point | Example |
|----------|-------------------|---------|
| **Build-time** | npm packages | Shared component library versioned as packages |
| **Server-side** | HTML stitching | SSI, ESI, Tailor, Podium |
| **Run-time (iframe)** | Isolated frames | Legacy integration; total isolation |
| **Run-time (JS)** | Module Federation / Import Maps | Webpack Module Federation, Native Federation |
| **Edge-side** | CDN composition | Edge-side includes, Cloudflare Workers |

### Communication Between MFEs

- **Custom Events**: Loosely coupled, pub/sub via `CustomEvent`.
- **Shared State Bus**: Lightweight event emitter or observable.
- **URL / Query Params**: For navigation-driven state sharing.
- **PostMessage**: For iframe-based isolation.

**Avoid**: Shared global stores — they create hidden coupling.

**Best for**: Large organizations, multiple autonomous teams, heterogeneous tech stacks.

---

## 3.4 Component-Driven Architecture

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

---

## 3.5 Jamstack / Edge Architecture

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

---

## 3.6 Hybrid / Transitional Architecture

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

Previous: [← Pattern Catalog](02-pattern-catalog.md) · Next: [Performance Architecture →](04-performance-architecture.md)
