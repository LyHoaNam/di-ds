# 4. Performance Architecture

[← Back to Index](frontend-engineering-framework.md)

---

## 4.1 Core Web Vitals — The North Star Metrics

| Metric | What it Measures | Target | Primary Lever |
|--------|-----------------|--------|---------------|
| **LCP** (Largest Contentful Paint) | Loading speed | ≤ 2.5s | Optimize critical resource delivery |
| **INP** (Interaction to Next Paint) | Responsiveness | ≤ 200ms | Reduce JS execution on main thread |
| **CLS** (Cumulative Layout Shift) | Visual stability | ≤ 0.1 | Reserve space for dynamic content |

---

## 4.2 Bundle Architecture

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

---

## 4.3 Network Performance

| Technique | Description |
|-----------|-------------|
| **CDN** | Serve static assets from edge locations nearest to user |
| **Compression** | Brotli (preferred) or Gzip for all text assets |
| **HTTP/2 & HTTP/3** | Multiplexing eliminates request queuing |
| **Preload / Prefetch** | `<link rel="preload">` for critical assets, `prefetch` for next-page assets |
| **Resource hints** | `preconnect`, `dns-prefetch` for third-party origins |
| **Service Workers** | Cache-first strategies for returning users; offline support |
| **Stale-While-Revalidate** | Serve from cache, revalidate in background |

---

## 4.4 Rendering Performance

- **Virtual DOM diffing**: Understand reconciliation — key props, avoiding unnecessary re-renders.
- **Memoization**: `React.memo`, `useMemo`, `useCallback`, `computed()` — use judiciously, not by default.
- **Virtualization**: Render only visible items in large lists/tables. Libraries: TanStack Virtual, react-window.
- **Debounce & Throttle**: Control rate of expensive operations (scroll, resize, input handlers).
- **Web Workers**: Off-load CPU-intensive tasks (parsing, compression, image processing).
- **requestAnimationFrame**: Synchronize visual updates with browser paint cycle.
- **CSS containment**: `contain: layout style paint` to isolate rendering subtrees.
- **Content-visibility**: `content-visibility: auto` to skip rendering offscreen content.

---

## 4.5 Asset Optimization

| Asset | Optimization Strategy |
|-------|----------------------|
| **Images** | Modern formats (WebP, AVIF), responsive `srcset`, lazy loading, CDN image transforms |
| **Fonts** | `font-display: swap`, subset only needed glyphs, preload critical fonts, self-host |
| **CSS** | Critical CSS inlined, remaining async-loaded, PurgeCSS for unused styles |
| **JavaScript** | Minification, scope hoisting, sideEffects flag, differential serving |
| **SVG** | Inline for icons, optimize with SVGO, use sprite sheets for many icons |

---

## 4.6 Performance Monitoring

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

**Performance budgets** — Set and enforce in CI:

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

Previous: [← Architecture Styles](03-architecture-styles.md) · Next: [Team Standards →](05-team-standards.md)
