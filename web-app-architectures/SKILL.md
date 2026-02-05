---
name: web-app-architectures
description: Explains Single Page Applications (SPA), Multi Page Applications (MPA), and hybrid architectures. Use when discussing app architecture decisions, comparing SPA vs MPA, explaining how traditional vs modern web apps work, or when building a new web application.
---

# Web Application Architectures

## Overview

Web applications fall into three main architectural patterns, each with distinct characteristics for navigation, state management, and user experience.

## Multi Page Application (MPA)

**The traditional web model** - each navigation triggers a full page request to the server.

### How It Works

```
User clicks link → Browser requests new HTML → Server renders full page → Browser loads entire page
```

### Characteristics

| Aspect | Behavior |
|--------|----------|
| Navigation | Full page reload on each route change |
| Initial Load | Fast - only current page HTML |
| Subsequent Navigation | Slower - full round trip |
| State | Lost on navigation (unless stored in cookies/sessions) |
| SEO | Excellent - each page is a complete HTML document |
| Server | Handles routing and rendering |

### When to Use MPA

- Content-heavy sites (blogs, documentation, news)
- SEO is critical
- Users have slow connections (less JS to download)
- Simple interactivity requirements
- Progressive enhancement is important

### Example Flow

```
/home → Server renders home.html
/about → Server renders about.html (full page reload)
/products/123 → Server renders product.html (full page reload)
```

## Single Page Application (SPA)

**The modern app model** - one HTML shell, JavaScript handles all routing and rendering.

### How It Works

```
Initial: Browser loads shell HTML + JS bundle
Navigation: JS intercepts clicks → Updates URL → Renders new view (no server request for HTML)
Data: Fetch API calls for JSON data only
```

### Characteristics

| Aspect | Behavior |
|--------|----------|
| Navigation | Instant (no page reload) |
| Initial Load | Slower - must download JS bundle |
| Subsequent Navigation | Fast - only fetch data, render client-side |
| State | Preserved across navigation |
| SEO | Challenging - requires additional strategies |
| Server | API endpoints only (JSON responses) |

### The SPA Tradeoffs

**Advantages:**
- App-like user experience
- Smooth transitions and animations
- Persistent UI state (music players, forms, etc.)
- Reduced server load after initial load

**Disadvantages:**
- Large initial JavaScript bundle
- SEO requires workarounds (SSR, prerendering)
- Memory management complexity
- Back button/deep linking need explicit handling
- Time to Interactive (TTI) can be slow

### Example Flow

```
/home → JS renders Home component
/about → JS renders About component (no server request)
/products/123 → JS fetches product data → Renders Product component
```

## Hybrid Architectures

Modern meta-frameworks blur the line between SPA and MPA.

### Multi Page App with Islands

MPA foundation with interactive "islands" of JavaScript.

```
Server renders full HTML → JS hydrates only interactive components
```

**Examples:** Astro, Fresh (Deno)

### SPA with Server-Side Rendering

SPA that pre-renders on server for initial load.

```
First request: Server renders full HTML + hydrates to SPA
Subsequent: Client-side navigation (SPA behavior)
```

**Examples:** Next.js, Nuxt, SvelteKit, Remix

### Streaming/Progressive Rendering

Server streams HTML as it becomes available.

```
Server starts sending HTML → Browser renders progressively → JS hydrates as content arrives
```

## Architecture Decision Matrix

| Requirement | Recommended |
|-------------|-------------|
| Content site, SEO critical | MPA or Hybrid with SSR |
| Dashboard, authenticated app | SPA or Hybrid |
| E-commerce (SEO + interactivity) | Hybrid with SSR |
| Minimal JS, fast initial load | MPA with Islands |
| Rich interactions, app-like UX | SPA or Hybrid |
| Limited team, simple stack | MPA |
| Offline support needed | SPA with Service Workers |

## Key Concepts to Understand

### Client-Side Routing

SPAs intercept navigation using the History API:

```javascript
// Instead of browser navigation
window.history.pushState({}, '', '/new-route');
// App renders new view without page reload
```

### Code Splitting

Breaking JS bundle into smaller chunks loaded on demand:

```
Initial: core.js (router, framework)
/dashboard: dashboard.chunk.js (loaded when needed)
/settings: settings.chunk.js (loaded when needed)
```

### State Persistence

SPAs maintain state in memory; MPAs must serialize state:

```
SPA: Component state survives navigation
MPA: State stored in URL params, cookies, localStorage, or server sessions
```

## Related Skills

- See [rendering-patterns](../rendering-patterns/SKILL.md) for SSR, SSG, CSR, ISR
- See [seo-fundamentals](../seo-fundamentals/SKILL.md) for SEO strategies
- See [hydration-patterns](../hydration-patterns/SKILL.md) for hydration concepts
- See [meta-frameworks-overview](../meta-frameworks-overview/SKILL.md) for framework comparisons
