---
name: hydration-patterns
description: Explains hydration, partial hydration, progressive hydration, and islands architecture. Use when discussing how server-rendered HTML becomes interactive, optimizing JavaScript loading, or choosing between full hydration and islands architecture.
---

# Hydration Patterns

## What is Hydration?

Hydration is the process of attaching JavaScript event handlers to server-rendered HTML, making static content interactive.

```
Server-Rendered HTML (static) → JavaScript Attaches → Interactive Application
        "Dry HTML"            →     Hydration      →    "Hydrated App"
```

## The Hydration Process

### Step by Step

```
1. Server renders HTML with content
2. Browser displays HTML immediately (fast FCP)
3. Browser downloads JavaScript bundle
4. JavaScript parses and executes
5. Framework "hydrates": 
   - Walks the DOM
   - Attaches event listeners
   - Initializes state
   - Makes components interactive
6. App is now fully interactive (TTI reached)
```

### The Hydration Gap

The time between content visible (FCP) and interactive (TTI):

```
Timeline:
[------- Server Render -------][-- HTML Sent --][---- JS Download ----][-- Hydration --]
                                                 ↓                                      ↓
                                            Content Visible                      Interactive
                                                 [========= Hydration Gap =========]
                                                 (Page looks ready but isn't)
```

## Hydration Challenges

### 1. Time to Interactive (TTI) Delay

Even though content is visible, buttons don't work until hydration completes.

```
User sees "Buy Now" button → Clicks → Nothing happens → Frustrated
                             (hydration not complete)
```

### 2. Hydration Mismatch

If client render differs from server render:

```javascript
// Server: <div>Today is Monday</div>
// Client (different timezone): <div>Today is Tuesday</div>
// Result: Error or UI flicker
```

### 3. Bundle Size

Full hydration requires downloading ALL component code:

```
Page uses: Header, Hero, ProductList, Footer, Cart, Reviews...
Bundle includes: ALL components (even if user never interacts)
```

### 4. CPU Cost

Hydration walks the entire DOM and attaches listeners:

```
Large page = More DOM nodes = Longer hydration = Worse TTI
```

## Full Hydration

**Traditional approach: hydrate the entire page.**

```javascript
// React example
import { hydrateRoot } from 'react-dom/client';
import App from './App';

// Hydrates entire app
hydrateRoot(document.getElementById('root'), <App />);
```

### Pros and Cons

| Pros | Cons |
|------|------|
| Simple mental model | Large JS bundles |
| Full interactivity everywhere | Slow TTI on large pages |
| Framework handles everything | Wasted JS for static content |

## Progressive Hydration

**Hydrate components in priority order, deferring non-critical parts.**

### Strategy

```
1. Hydrate above-the-fold content first
2. Hydrate below-the-fold on scroll or idle
3. Hydrate low-priority on user interaction
```

### Implementation Pattern

```javascript
// Conceptual - defer hydration until visible
function LazyHydrate({ children, whenVisible }) {
  const [hydrated, setHydrated] = useState(false);
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setHydrated(true);
      }
    });
    observer.observe(ref.current);
    return () => observer.disconnect();
  }, []);
  
  if (!hydrated) return <div ref={ref}>{/* static HTML */}</div>;
  return children;
}
```

### Timeline

```
[Initial Hydration - Critical Components Only]
        ↓
[Scroll/Idle - Hydrate More Components]
        ↓
[Interaction - Hydrate On Demand]
```

## Selective Hydration

**Only hydrate components that need interactivity.**

React 18+ with Suspense enables this:

```jsx
<Suspense fallback={<Loading />}>
  <Comments /> {/* Hydrates independently when ready */}
</Suspense>
```

### How It Works

```
Server streams: Header (hydrated) → Content (hydrated) → Comments (pending)
Comments data arrives → Comments hydrate independently
User clicks comment before hydration → React prioritizes that subtree
```

## Partial Hydration

**Ship zero JavaScript for static components.**

### The Insight

Most pages have static and interactive parts:

```
┌─────────────────────────────────────┐
│  Header (static)                    │ ← No JS needed
├─────────────────────────────────────┤
│  Hero Banner (static)               │ ← No JS needed
├─────────────────────────────────────┤
│  Product List (static)              │ ← No JS needed
├─────────────────────────────────────┤
│  Add to Cart Button ★               │ ← NEEDS JS
├─────────────────────────────────────┤
│  Reviews (static)                   │ ← No JS needed
├─────────────────────────────────────┤
│  Newsletter Signup ★                │ ← NEEDS JS
├─────────────────────────────────────┤
│  Footer (static)                    │ ← No JS needed
└─────────────────────────────────────┘
```

**With partial hydration:** Only "Add to Cart" and "Newsletter" components ship JS.

## Islands Architecture

**Independent interactive components in a sea of static HTML.**

### Concept

```
┌─────────────────────────────────────────────────┐
│                                                 │
│   Static HTML (server-rendered, no JS)          │
│                                                 │
│   ┌─────────────┐          ┌─────────────┐      │
│   │   Island    │          │   Island    │      │
│   │  (Counter)  │          │  (Search)   │      │
│   │   [JS: 2KB] │          │   [JS: 5KB] │      │
│   └─────────────┘          └─────────────┘      │
│                                                 │
│   Static content continues...                   │
│                                                 │
│   ┌─────────────────────────────────────┐      │
│   │           Island (Comments)          │      │
│   │              [JS: 8KB]               │      │
│   └─────────────────────────────────────┘      │
│                                                 │
└─────────────────────────────────────────────────┘

Total JS: 15KB (vs potentially 200KB+ for full hydration)
```

### Key Characteristics

| Aspect | Full Hydration | Islands |
|--------|----------------|---------|
| JS shipped | All components | Only interactive |
| Hydration | Entire page | Per island |
| Bundle size | Large | Minimal |
| TTI | Slow | Fast |
| Coupling | Components connected | Islands independent |

### Frameworks Using Islands

- **Astro:** First-class islands support
- **Fresh (Deno):** Islands by default
- **Îles:** Islands for Vite
- **Qwik:** Resumability (related concept)

### Island Definition Example (Astro)

```astro
---
// Static by default
import Header from './Header.astro';
import Counter from './Counter.jsx';  // React component
---

<Header />  <!-- No JS shipped -->

<Counter client:load />  <!-- JS shipped, hydrates on load -->

<Counter client:visible />  <!-- JS shipped, hydrates when visible -->

<Counter client:idle />  <!-- JS shipped, hydrates on browser idle -->
```

## Resumability (Qwik's Approach)

**Skip hydration entirely by serializing application state.**

### Traditional Hydration

```
Server renders → Client downloads JS → Re-executes to rebuild state → Attaches listeners
```

### Resumability

```
Server renders + serializes state → Client downloads minimal JS → Resumes exactly where server left off
```

### Key Difference

```javascript
// Hydration: Re-run component code to figure out handlers
// Resumability: Handlers serialized in HTML, just attach them

// Qwik serializes even closures:
<button on:click="./chunk.js#handleClick[0]">Click me</button>
```

## Choosing a Hydration Strategy

| Scenario | Recommended Pattern |
|----------|---------------------|
| SPA/Dashboard | Full hydration |
| Content site with some interactivity | Islands |
| E-commerce product page | Partial/Progressive |
| Marketing landing page | Islands or SSG (no hydration) |
| Highly interactive app | Full hydration with code splitting |
| Performance critical, varied interactivity | Islands or Resumability |

## Hydration Anti-Patterns

### 1. Hydration Mismatch

```javascript
// Bad: Different output on server vs client
function Greeting() {
  return <p>Hello at {new Date().toLocaleTimeString()}</p>;
}

// Good: Defer client-only values
function Greeting() {
  const [time, setTime] = useState(null);
  useEffect(() => setTime(new Date().toLocaleTimeString()), []);
  return <p>Hello{time && ` at ${time}`}</p>;
}
```

### 2. Blocking Hydration on Data

```javascript
// Bad: Hydration waits for fetch
useEffect(() => {
  fetchData().then(setData);  // Delays hydration
}, []);

// Good: Use server-provided data or streaming
```

### 3. Huge Hydration Bundles

```javascript
// Bad: Import everything at top level
import { FullCalendar } from 'massive-calendar-lib';

// Good: Dynamic import for heavy components
const Calendar = lazy(() => import('./Calendar'));
```

## Related Skills

- See [rendering-patterns](../rendering-patterns/SKILL.md) for SSR/SSG context
- See [web-app-architectures](../web-app-architectures/SKILL.md) for SPA vs MPA
- See [meta-frameworks-overview](../meta-frameworks-overview/SKILL.md) for framework hydration strategies
