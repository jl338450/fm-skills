---
name: seo-fundamentals
description: Explains SEO for web applications including crawling, indexing, Core Web Vitals, structured data, and SEO challenges with SPAs. Use when optimizing for search engines, discussing SEO implications of architecture decisions, or implementing SEO best practices.
---

# SEO Fundamentals for Web Applications

## Overview

SEO (Search Engine Optimization) ensures search engines can discover, understand, and rank your content. Architecture decisions significantly impact SEO capability.

## How Search Engines Work

### The Three Phases

```
1. CRAWLING: Googlebot discovers URLs and downloads content
2. INDEXING: Google parses content and stores in search index
3. RANKING: Algorithm determines position in search results
```

### What Crawlers See

Crawlers primarily process the **initial HTML response**. JavaScript execution is:
- Delayed (crawl queue, separate rendering queue)
- Resource-intensive (limited render budget)
- Not guaranteed for all pages

```
Your Server Response          What Googlebot Sees First
─────────────────────         ──────────────────────────
<!DOCTYPE html>               Same HTML (good for SSR/SSG)
<html>
<body>                        OR
  <div id="root"></div>       Empty div (bad for CSR)
  <script src="app.js">       Script tag (won't execute immediately)
</body>
</html>
```

## Architecture Impact on SEO

### SEO by Rendering Pattern

| Pattern | Initial HTML | SEO Quality | Notes |
|---------|--------------|-------------|-------|
| SSG | Complete | Excellent | Best for SEO |
| SSR | Complete | Excellent | Dynamic content, good SEO |
| ISR | Complete | Excellent | Fresh + fast |
| CSR | Empty shell | Poor | Requires workarounds |
| Streaming | Progressive | Good | Shell + streamed content |

### The CSR Problem

```javascript
// What your React app renders client-side
<html>
  <head><title>My App</title></head>
  <body>
    <div id="root">
      <!-- JS renders content here AFTER page load -->
      <!-- Crawler may not wait for this -->
    </div>
  </body>
</html>
```

### Solutions for CSR Apps

1. **Server-Side Rendering (SSR):** Render on server, hydrate on client
2. **Pre-rendering:** Generate static HTML at build time for key pages
3. **Dynamic Rendering:** Serve pre-rendered HTML to bots, SPA to users (not recommended by Google)

## Technical SEO Elements

### Essential Meta Tags

```html
<head>
  <!-- Title: 50-60 characters, unique per page -->
  <title>Product Name - Category | Brand</title>
  
  <!-- Description: 150-160 characters -->
  <meta name="description" content="Compelling description with keywords">
  
  <!-- Canonical: Prevent duplicate content -->
  <link rel="canonical" href="https://example.com/page">
  
  <!-- Robots: Control indexing -->
  <meta name="robots" content="index, follow">
  
  <!-- Open Graph: Social sharing -->
  <meta property="og:title" content="Page Title">
  <meta property="og:description" content="Description">
  <meta property="og:image" content="https://example.com/image.jpg">
  
  <!-- Viewport: Mobile-friendliness -->
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
```

### Structured Data (JSON-LD)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "description": "Product description",
  "price": "29.99",
  "priceCurrency": "USD",
  "availability": "https://schema.org/InStock"
}
</script>
```

Common schema types: Product, Article, FAQ, BreadcrumbList, Organization, LocalBusiness

### Semantic HTML

```html
<!-- Good: Semantic structure -->
<main>
  <article>
    <header>
      <h1>Main Title</h1>
    </header>
    <section>
      <h2>Section Title</h2>
      <p>Content...</p>
    </section>
  </article>
  <aside>Related content</aside>
</main>

<!-- Bad: Div soup -->
<div class="main">
  <div class="title">Main Title</div>
  <div class="content">Content...</div>
</div>
```

## Core Web Vitals

Google's page experience metrics that affect ranking.

### The Three Metrics

| Metric | What It Measures | Good | Needs Improvement | Poor |
|--------|------------------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | Loading performance | ≤2.5s | 2.5-4s | >4s |
| **INP** (Interaction to Next Paint) | Interactivity | ≤200ms | 200-500ms | >500ms |
| **CLS** (Cumulative Layout Shift) | Visual stability | ≤0.1 | 0.1-0.25 | >0.25 |

### Common Issues by Architecture

**SPA/CSR Issues:**
- LCP: Slow due to JS loading before content
- CLS: Content shifts as JS loads and renders

**SSR Issues:**
- INP: Hydration can block interactivity
- TTFB: Slow server response times

**SSG Issues:**
- Generally best for Core Web Vitals
- CLS: Still possible with lazy-loaded images

### Optimization Strategies

**Improve LCP:**
```html
<!-- Preload critical resources -->
<link rel="preload" href="hero-image.jpg" as="image">
<link rel="preload" href="critical.css" as="style">

<!-- Inline critical CSS -->
<style>/* Above-the-fold styles */</style>

<!-- Defer non-critical JS -->
<script defer src="app.js"></script>
```

**Improve CLS:**
```html
<!-- Reserve space for images -->
<img src="photo.jpg" width="800" height="600" alt="Description">

<!-- Reserve space for ads/embeds -->
<div style="min-height: 250px;">
  <!-- Ad will load here -->
</div>
```

**Improve INP:**
- Break up long tasks
- Minimize hydration cost
- Use `requestIdleCallback` for non-critical work

## URL Structure

### Best Practices

```
Good URLs:
/products/blue-running-shoes
/blog/2024/seo-guide
/category/electronics/phones

Poor URLs:
/products?id=12345
/page.php?cat=1&sub=2
/p/abc123xyz
```

### URL Guidelines

- Use hyphens, not underscores
- Keep URLs short and descriptive
- Include target keywords naturally
- Use lowercase only
- Avoid query parameters for indexable content

## Sitemap and Robots.txt

### XML Sitemap

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/page</loc>
    <lastmod>2024-01-15</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

### Robots.txt

```
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/
Sitemap: https://example.com/sitemap.xml
```

## SPA SEO Checklist

For SPAs that need SEO:

- [ ] Implement SSR or pre-rendering for SEO-critical pages
- [ ] Ensure each route has unique meta tags (title, description)
- [ ] Use semantic HTML structure
- [ ] Implement proper heading hierarchy (h1 → h2 → h3)
- [ ] Add structured data (JSON-LD)
- [ ] Generate XML sitemap with all routes
- [ ] Handle redirects server-side (301/302), not client-side
- [ ] Implement canonical URLs
- [ ] Ensure internal links are crawlable `<a href>` tags
- [ ] Test with Google Search Console's URL Inspection tool

## Common SEO Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Client-side only meta tags | Crawler sees defaults | SSR or head management |
| JavaScript-only navigation | Links not crawlable | Use `<a href>` tags |
| Infinite scroll | Content not discoverable | Pagination or "Load More" with URLs |
| Hash-based routing | URLs not indexed | Use History API (`/path` not `/#/path`) |
| Duplicate content | Diluted rankings | Canonical tags |
| Slow loading | Poor rankings | Optimize Core Web Vitals |

## Testing SEO

### Tools

- **Google Search Console:** Indexing status, issues
- **Lighthouse:** Core Web Vitals, SEO audit
- **View Page Source:** What crawler sees (not DevTools)
- **Google Rich Results Test:** Structured data validation

### Quick Checks

```bash
# See what Google sees
curl -A "Googlebot" https://example.com/page

# Check robots.txt
curl https://example.com/robots.txt

# View without JavaScript (approximate crawler view)
# Disable JavaScript in browser DevTools
```

## Related Skills

- See [web-app-architectures](../web-app-architectures/SKILL.md) for SPA vs MPA
- See [rendering-patterns](../rendering-patterns/SKILL.md) for SSR, SSG impact
- See [meta-frameworks-overview](../meta-frameworks-overview/SKILL.md) for built-in SEO features
