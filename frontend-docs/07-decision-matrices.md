# Quick Reference: Decision Matrices

[← Back to Index](frontend-engineering-framework.md)

---

## When to Use What Rendering Strategy

```
Is the content public & SEO-critical?
├── Yes → Does it change rarely?
│         ├── Yes → SSG / ISR
│         └── No  → SSR (streaming)
└── No  → Is it a highly interactive app?
          ├── Yes → CSR (SPA) with code splitting
          └── No  → SSR or hybrid
```

---

## When to Use What State Management

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

---

## When to Adopt Micro-Frontends

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

Previous: [← Learning Roadmap](06-learning-roadmap.md) · Back to: [Index](frontend-engineering-framework.md)
