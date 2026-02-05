---
name: rendering-patterns
description: Explains web rendering strategies including Client-Side Rendering (CSR), Server-Side Rendering (SSR), Static Site Generation (SSG), Incremental Static Regeneration (ISR), and streaming. Use when deciding rendering strategy, optimizing performance, or understanding when content is generated and sent to users.
---

# Web Rendering Patterns

## Overview

Rendering determines **when** and **where** HTML is generated. Each pattern has distinct performance, SEO, and infrastructure implications.

## Rendering Pattern Summary

| Pattern | When Generated | Where Generated | Use Case |
|---------|----------------|-----------------|----------|
| CSR | Runtime (browser) | Client | Dashboards, authenticated apps |
| SSR | Runtime (each request) | Server | Dynamic, personalized content |
| SSG | Build time | Server/Build | Static content, blogs, docs |
| ISR | Build + revalidation | Server | Content that changes periodically |
| Streaming | Runtime (progressive) | Server | Long pages, slow data sources |

## Client-Side Rendering (CSR)

**Browser generates HTML using JavaScript after page load.**

### Flow

```
1. Browser requests page
2. Server returns minimal HTML shell + JS bundle
3. JS executes in browser
4. JS fetches data from API
5. JS renders HTML into DOM
```

### Timeline

```
Request → Empty Shell → JS Downloads → JS Executes → Data Fetches → Content Visible
         [--------------- Time to Interactive (slow) ---------------]
```

### Characteristics

- **First Contentful Paint (FCP):** Slow (waiting for JS)
- **Time to Interactive (TTI):** Slow (JS must execute + fetch data)
- **SEO:** Poor (crawlers see empty shell unless they execute JS)
- **Server load:** Low (static files only)
- **Caching:** Easy (static assets)

### When to Use

- Authenticated dashboards (no SEO needed)
- Highly interactive apps
- Real-time data that can't be pre-rendered
- When server infrastructure is limited

### Code Pattern

```jsx
// Pure CSR - data fetched in browser
function ProductPage() {
  const [product, setProduct] = useState(null);
  
  useEffect(() => {
    fetch('/api/products/123')
      .then(res => res.json())
      .then(setProduct);
  }, []);
  
  if (!product) return <Loading />;
  return <Product data={product} />;
}
```

## Server-Side Rendering (SSR)

**Server generates complete HTML for each request.**

### Flow

```
1. Browser requests page
2. Server fetches data
3. Server renders HTML with data
4. Server sends complete HTML
5. Browser displays content immediately
6. JS hydrates for interactivity
```

### Timeline

```
Request → Server Fetches Data → Server Renders → HTML Sent → Content Visible → Hydration → Interactive
          [--- Server Time ---]                              [--- Hydration ---]
```

### Characteristics

- **FCP:** Fast (complete HTML from server)
- **TTI:** Depends on hydration time
- **SEO:** Excellent (full HTML for crawlers)
- **Server load:** High (render on every request)
- **Caching:** Complex (varies by user/request)

### When to Use

- SEO-critical pages with dynamic content
- Personalized content (user-specific)
- Frequently changing data
- Pages that need fresh data on every request

### Code Pattern (Framework-agnostic concept)

```javascript
// Server-side: runs on each request
async function renderPage(request) {
  const data = await fetchData(request.params.id);
  const html = renderToString(<Page data={data} />);
  return html;
}
```

## Static Site Generation (SSG)

**HTML generated once at build time.**

### Flow

```
Build Time:
1. Build process fetches all data
2. Generates HTML for all pages
3. Outputs static files

Runtime:
1. Browser requests page
2. CDN serves pre-built HTML instantly
```

### Timeline

```
Request → CDN Cache Hit → HTML Delivered → Content Visible (instant)
```

### Characteristics

- **FCP:** Fastest (pre-built, CDN-cached)
- **TTI:** Fast (minimal JS, or none)
- **SEO:** Excellent (complete HTML)
- **Server load:** None at runtime (static files)
- **Caching:** Trivial (immutable until next build)

### When to Use

- Blogs, documentation, marketing pages
- Content that changes infrequently
- Pages where all possible URLs are known at build time
- Maximum performance is required

### Limitations

- Content stale until rebuild
- Build time grows with page count
- Can't handle dynamic routes unknown at build time
- Personalization requires client-side hydration

## Incremental Static Regeneration (ISR)

**SSG with automatic background regeneration.**

### Flow

```
1. Initial build generates static pages
2. Pages served from cache
3. After revalidation time expires:
   - Serve stale page (fast)
   - Regenerate in background
   - Next request gets fresh page
```

### Revalidation Strategies

**Time-based:**
```
Page generated → Serve for 60 seconds → Regenerate on next request after 60s
```

**On-demand:**
```
CMS publishes → Webhook triggers regeneration → Page updated immediately
```

### Characteristics

- **FCP:** Fast (cached HTML)
- **Freshness:** Configurable (seconds to hours)
- **SEO:** Excellent
- **Server load:** Low (regenerate occasionally)
- **Scalability:** High (mostly static)

### When to Use

- E-commerce product pages
- News sites
- Content that updates but not real-time
- High-traffic pages that need freshness

## Streaming / Progressive Rendering

**Server sends HTML in chunks as data becomes available.**

### Flow

```
1. Browser requests page
2. Server immediately sends HTML shell
3. Server fetches data (possibly in parallel)
4. Server streams HTML chunks as ready
5. Browser renders progressively
```

### Timeline

```
Request → Shell Sent → [Chunk 1 Streams] → [Chunk 2 Streams] → Complete
          [Visible]    [More Visible]      [Fully Visible]
```

### Characteristics

- **FCP:** Very fast (shell immediate)
- **TTFB:** Fast (no waiting for all data)
- **User Experience:** Progressive disclosure
- **Complexity:** Higher implementation

### When to Use

- Pages with multiple data sources
- Slow API dependencies
- Long pages where top should render first
- Improving perceived performance

## Rendering Decision Flowchart

```
Does the page need SEO?
├── No → Does it need real-time data?
│        ├── Yes → CSR
│        └── No → SSG or CSR
│
└── Yes → Is content the same for all users?
          ├── Yes → Does content change often?
          │         ├── Rarely → SSG
          │         ├── Sometimes → ISR
          │         └── Every request → SSR
          │
          └── No (personalized) → SSR with caching strategies
```

## Hybrid Approaches

Modern apps mix patterns per route:

| Route | Pattern | Reason |
|-------|---------|--------|
| `/` (home) | SSG | Static marketing content |
| `/blog/*` | SSG/ISR | Content changes occasionally |
| `/products/*` | ISR | Prices update, need SEO |
| `/dashboard` | CSR | Authenticated, no SEO |
| `/search` | SSR | Dynamic query results |

## Performance Metrics Impact

| Pattern | TTFB | FCP | LCP | TTI |
|---------|------|-----|-----|-----|
| CSR | Fast | Slow | Slow | Slow |
| SSR | Slower | Fast | Fast | Medium |
| SSG | Fastest | Fastest | Fastest | Fast |
| ISR | Fastest | Fastest | Fastest | Fast |
| Streaming | Fast | Fast | Progressive | Medium |

## Related Skills

- See [web-app-architectures](../web-app-architectures/SKILL.md) for SPA vs MPA
- See [seo-fundamentals](../seo-fundamentals/SKILL.md) for SEO implications
- See [hydration-patterns](../hydration-patterns/SKILL.md) for hydration strategies
