---
name: meta-frameworks-overview
description: Explains meta-frameworks like Next.js, Nuxt, SvelteKit, Remix, Astro, and their architectural patterns. Use when comparing frameworks, choosing a framework for a project, or understanding what features meta-frameworks provide beyond base UI libraries.
---

# Meta-Frameworks Overview

## What is a Meta-Framework?

A meta-framework builds on top of a UI library (React, Vue, Svelte) to provide:

```
UI Library (React, Vue, Svelte)
    ↓ adds
Meta-Framework Features:
├── File-based routing
├── Server-side rendering (SSR)
├── Static site generation (SSG)
├── API routes / backend integration
├── Code splitting & bundling
├── Image optimization
├── Deployment optimizations
└── Full-stack capabilities
```

## Why Meta-Frameworks?

### Without a Meta-Framework (Manual Setup)

```
Create React App
├── Configure Webpack/Vite
├── Set up React Router
├── Configure SSR manually (complex)
├── Set up Express/Node server
├── Configure code splitting
├── Set up environment variables
├── Configure production build
└── Handle deployment
```

### With a Meta-Framework

```
npx create-next-app / npm create astro
├── Routing: ✓ automatic
├── SSR/SSG: ✓ built-in
├── API routes: ✓ included
├── Bundling: ✓ optimized
├── Deployment: ✓ streamlined
└── Start building: ✓ immediately
```

## Framework Comparison Matrix

| Framework | UI Library | Rendering | Philosophy |
|-----------|------------|-----------|------------|
| **Next.js** | React | SSR, SSG, ISR, CSR | Full-stack React |
| **Remix** | React | SSR, progressive enhancement | Web standards, forms |
| **Nuxt** | Vue | SSR, SSG, ISR | Full-stack Vue |
| **SvelteKit** | Svelte | SSR, SSG, CSR | Compiler-first |
| **Astro** | Any (React, Vue, Svelte, etc.) | SSG, SSR, Islands | Content-first, minimal JS |
| **Solid Start** | Solid | SSR, SSG | Fine-grained reactivity |
| **Qwik City** | Qwik | Resumable | Zero hydration |

## Next.js

**Full-stack React framework by Vercel.**

### Key Features

- **App Router** (new): React Server Components, nested layouts
- **Pages Router** (legacy): Traditional file-based routing
- **Rendering options:** SSG, SSR, ISR, CSR per route
- **API Routes:** Backend endpoints within the same project
- **Image/Font optimization:** Built-in components

### Routing Model

```
app/
├── page.tsx          → /
├── about/page.tsx    → /about
├── blog/
│   ├── page.tsx      → /blog
│   └── [slug]/page.tsx → /blog/:slug
└── api/
    └── route.ts      → API endpoint
```

### Rendering Decision

```tsx
// Static (default - SSG)
export default function Page() {
  return <div>Static content</div>;
}

// Dynamic (SSR)
export const dynamic = 'force-dynamic';

// ISR
export const revalidate = 60; // seconds
```

### Best For

- Production React applications
- E-commerce, SaaS, marketing sites
- Teams wanting batteries-included solution
- Vercel deployment (optimized)

## Remix

**Full-stack React framework focused on web fundamentals.**

### Key Features

- **Nested routes:** Parallel data loading
- **Progressive enhancement:** Works without JavaScript
- **Form handling:** Built-in `<Form>` with server actions
- **Error boundaries:** Per-route error handling
- **Web standards:** Uses fetch, FormData, Response

### Philosophy

```
Traditional SPA:
  Click → JS preventDefault → Fetch API → Update State → Re-render

Remix:
  Click → Form submits → Server action → Return data → Re-render
  (Works without JS, enhanced with JS)
```

### Routing Model

```
app/
├── routes/
│   ├── _index.tsx        → /
│   ├── about.tsx         → /about
│   ├── blog._index.tsx   → /blog
│   └── blog.$slug.tsx    → /blog/:slug
└── root.tsx              → Root layout
```

### Data Loading

```tsx
// Parallel loading of nested route data
export async function loader({ params }) {
  return json(await getPost(params.slug));
}

export async function action({ request }) {
  const formData = await request.formData();
  // Handle form submission
}
```

### Best For

- Form-heavy applications
- Progressive enhancement requirements
- Teams preferring web standards
- Apps that should work without JS

## Nuxt

**Full-stack Vue framework.**

### Key Features

- **Auto-imports:** Components, composables auto-imported
- **File-based routing:** Vue components as routes
- **Nitro server:** Universal deployment
- **Modules ecosystem:** Rich plugin system

### Routing Model

```
pages/
├── index.vue         → /
├── about.vue         → /about
└── blog/
    ├── index.vue     → /blog
    └── [slug].vue    → /blog/:slug
```

### Data Fetching

```vue
<script setup>
// Runs on server, cached
const { data } = await useFetch('/api/posts');

// Always fresh
const { data } = await useFetch('/api/posts', { fresh: true });
</script>
```

### Best For

- Vue.js teams
- Content sites, applications
- Teams wanting Vue-specific optimizations

## SvelteKit

**Full-stack Svelte framework.**

### Key Features

- **Compiler-first:** Svelte compiles to vanilla JS
- **Adapters:** Deploy anywhere (Node, Vercel, Cloudflare, etc.)
- **Load functions:** Unified data loading
- **Form actions:** Built-in form handling

### Routing Model

```
src/routes/
├── +page.svelte          → /
├── +layout.svelte        → Shared layout
├── about/+page.svelte    → /about
└── blog/
    ├── +page.svelte      → /blog
    └── [slug]/+page.svelte → /blog/:slug
```

### Data Loading

```javascript
// +page.server.js - runs on server only
export async function load({ params }) {
  return {
    post: await getPost(params.slug)
  };
}
```

### Best For

- Performance-critical applications
- Teams preferring Svelte's approach
- Smaller bundle size requirements

## Astro

**Content-first framework with islands architecture.**

### Key Features

- **Zero JS by default:** Ships no JavaScript unless needed
- **Islands architecture:** Hydrate only interactive components
- **Framework agnostic:** Use React, Vue, Svelte together
- **Content collections:** First-class Markdown/MDX support

### Routing Model

```
src/pages/
├── index.astro           → /
├── about.astro           → /about
└── blog/
    ├── index.astro       → /blog
    └── [slug].astro      → /blog/:slug
```

### Islands Pattern

```astro
---
import Header from './Header.astro';     // No JS
import Counter from './Counter.jsx';      // React
import Search from './Search.vue';        // Vue
---

<Header />

<!-- Islands: only these ship JS -->
<Counter client:load />
<Search client:visible />
```

### Client Directives

| Directive | When Hydrates |
|-----------|---------------|
| `client:load` | Immediately on page load |
| `client:idle` | When browser is idle |
| `client:visible` | When component is visible |
| `client:media` | When media query matches |
| `client:only` | Skip SSR, client render only |

### Best For

- Content-heavy sites (blogs, docs, marketing)
- Performance-first approach
- Teams using multiple UI frameworks
- Static sites with some interactivity

## Qwik City

**Framework with resumability (zero hydration).**

### Key Features

- **Resumability:** No hydration cost
- **Lazy loading:** Loads JS only when needed
- **Familiar syntax:** JSX-like templates

### How Resumability Works

```
Traditional SSR:
  Server render → Download all JS → Hydrate (re-execute) → Interactive

Qwik:
  Server render + serialize state → Load tiny runtime → Resume → Interactive
  (No re-execution, state already in HTML)
```

### Best For

- Performance-critical applications
- Large applications with many components
- Teams wanting to eliminate hydration cost

## Framework Selection Guide

### By Project Type

| Project Type | Recommended |
|--------------|-------------|
| Marketing site | Astro, Next.js (SSG) |
| Blog/Docs | Astro, Next.js, SvelteKit |
| E-commerce | Next.js, Remix, Nuxt |
| Dashboard/App | Next.js, SvelteKit, Remix |
| Highly interactive app | Next.js, SvelteKit, Remix |
| Performance-critical | Astro, Qwik, SvelteKit |

### By Team Expertise

| Team Knows | Recommended |
|------------|-------------|
| React | Next.js, Remix, Astro |
| Vue | Nuxt, Astro |
| Svelte | SvelteKit, Astro |
| Multiple / Learning | Astro |

### By Priorities

| Priority | Recommended |
|----------|-------------|
| SEO | Any (all support SSR/SSG) |
| Minimal JavaScript | Astro, Qwik |
| Progressive enhancement | Remix |
| Largest ecosystem | Next.js |
| Fastest runtime | SvelteKit, Qwik |
| Most flexible | Astro |

## Common Patterns Across Frameworks

### File-Based Routing

All meta-frameworks use file structure for routes:

```
pages/about.tsx  →  /about
pages/[id].tsx   →  /123, /abc (dynamic)
pages/[...slug].tsx  →  /a/b/c (catch-all)
```

### Layouts

Shared UI across routes:

```
layout.tsx (Next.js)
+layout.svelte (SvelteKit)
layouts/default.vue (Nuxt)
```

### Data Loading

Fetching data before render:

```
getServerSideProps / loader functions (Next.js)
loader functions (Remix)
load functions (SvelteKit)
useFetch (Nuxt)
frontmatter + getStaticProps (Astro)
```

### API Routes

Backend endpoints in the same project:

```
app/api/route.ts (Next.js)
app/routes/api.$.ts (Remix)
server/api/ (Nuxt)
src/pages/api/ (Astro)
```

## Migration Considerations

### From SPA to Meta-Framework

1. Choose framework matching your UI library
2. Migrate routing to file-based
3. Move data fetching to server (loaders)
4. Update build/deploy pipeline
5. Add SSR/SSG as needed

### Between Meta-Frameworks

- Routing patterns are similar (map file structure)
- Data fetching patterns differ (loaders, getServerSideProps, useFetch)
- Styling approaches may vary
- Deployment adapters differ

## Related Skills

- See [web-app-architectures](../web-app-architectures/SKILL.md) for SPA vs MPA
- See [rendering-patterns](../rendering-patterns/SKILL.md) for SSR/SSG/ISR
- See [hydration-patterns](../hydration-patterns/SKILL.md) for hydration strategies
- See [routing-patterns](../routing-patterns/SKILL.md) for routing concepts
