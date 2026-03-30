# 5. Team Standards

[← Back to Index](frontend-engineering-framework.md)

---

## 5.1 Code Quality Toolchain

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

---

## 5.2 TypeScript Standards

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

---

## 5.3 Testing Strategy — The Testing Trophy

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

---

## 5.4 CSS / Styling Architecture

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

---

## 5.5 Git & Branching Strategy

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

---

## 5.6 Code Review Standards

| Area | What to Check |
|------|---------------|
| **Correctness** | Does it do what it claims? Edge cases handled? |
| **Security** | XSS vectors, auth checks, sensitive data exposure, dependency risks |
| **Performance** | Unnecessary re-renders, large bundles, unoptimized assets |
| **Accessibility** | Semantic HTML, ARIA, keyboard navigation, color contrast |
| **Readability** | Clear naming, reasonable abstractions, not over-engineered |
| **Tests** | Meaningful coverage for new logic; no snapshot-only tests |
| **Types** | No `any` escapes, proper null handling, discriminated unions |

---

## 5.7 Accessibility (a11y) Standards

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

---

## 5.8 Documentation Standards

| Artifact | Tool / Format | Purpose |
|----------|---------------|---------|
| **Component catalog** | Storybook, Histoire, Ladle | Interactive showcase of all UI components |
| **API docs** | TypeDoc, TSDoc comments | Auto-generated from TypeScript types |
| **ADRs** | Markdown in `docs/adr/` | Record architectural decisions and rationale |
| **Runbooks** | Markdown / Wiki | How to deploy, roll back, debug production |
| **README** | Markdown | Setup, architecture overview, conventions |
| **Changelog** | Auto-generated | Track user-facing changes per release |

---

## 5.9 Monorepo Architecture

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

Previous: [← Performance Architecture](04-performance-architecture.md) · Next: [Learning Roadmap →](06-learning-roadmap.md)
