<h1 align="center">
  STAX-stack
</h1>

<p align="center">
  Specification and standardization of a modular web stack.
</p>

## Contents

- [Axioms](#--axoims)
- [Architecture Overview](--#architecture-overview)
- [Decision Tree](#--decision-tree)
- [Technology Overview](#--technology-overview)
    - [Client Layer](#client-layer)
    - [API Layer](#api-layer)
    - [Data Layer](#data-layer)
- [Detailed Scenarios](#detailed-scenarios)

<h2 align="center">
  Axioms
</h2>

This stack is built on four core principles:

1. **Low Complexity**: Each tool does one thing well. No framework sprawl, no over-engineering. Choose the simplest solution for each problem domain.
2. **High Performance**: Edge-native architecture with minimal to no JavaScript, aggressive cache capabilities, reduced distribution size, minimal hydration, minimal to no double rendering, and compiled ahead-of-time components. Every technology choice optimizes for speed.
3. **Full Situational Coverage**: Different problems need different solutions. This stack provides the right tool for static content, dynamic UIs, server interactions, and public APIs without forcing everything through one paradigm. And every component of the tech stack is compatible and complements the next.
4. **High Robustness**: Type safety across client-server boundaries, progressive enhancement, and graceful degradation. Works without JavaScript where possible, easy to prototype with, and scales to elastically to high demand.

<h2 align="center">
  Architecture Overview
</h2>

The architecture is broken into three layers:
1. Client Layer
    - Rendering UI
    - Client-side interactivity
2. API Layer
    - Client-server communication
    - Public API
3. Data Layer
    - Store data close to client
    - Store objects to be fetched

```
┌──────────────────────────────────────────────┐
│               CLIENT LAYER                   │
│                                              │
│  ┌──────────────────┐  ┌──────────────────┐  │
│  │ TypeScript       │  │ TailwindCSS      │  │
│  └─────────┬────────┘  └─────────┬────────┘  │
│            ├─────────────────────┤           │
│            │                     │           │
│            │                     │           │
│  ┌─────────▼────────┐  ┌─────────▼────────┐  │
│  │ Astro Static     │  │ Astro Island(s)  │  │
│  │ HTML Components  │  │ (Svelte)         │  │
│  │ (No JS)          │  │                  │  │
│  └─────────▲────────┘  └─────▲──────▲─────┘  │
└────────────│─────────────────│──────│────────┘
             │                 │      │
             ├─────────────────┘      │            PUBLIC API
             │                        │                ▲
             │                        │                │
┌────────────│────────────────────────│────────────────│──────────┐
│            │              API LAYER │                │          │
│            │                        │                │          │
│  ┌─────────▼──────┐  ┌──────────────▼─┐  ┌───────────▼───────┐  │
│  │ HTMX Worker(s) │  │ tRPC Worker(s) │  │ GraphQL Worker(s) │  │
│  │ (Rust)         │  │ (TypeScript)   │  │ (Rust)            │  │
│  │ Internal       │  │ Internal       │  │ Public            │  │
│  │ Returns: HTML  │  │ Returns: JSON  │  │ Returns: JSON     │  │
│  └────────▲───────┘  └───────▲────────┘  └─────────▲─────────┘  │
└───────────│──────────────────│─────────────────────│────────────┘
            │                  │                     │
            └───┬──────────────┴──────────┬──────────┘
                │                         │
                │                         │
┌───────────────│─────────────────────────│───────────────┐
│               │      DATA LAYER         │               │
│               │                         │               │
│    ┌──────────▼─────────┐     ┌─────────▼────────┐      │
│    │  Edge Database(s)  │     │  Object Storage  │      │
│    └────────────────────┘     └──────────────────┘      │
└─────────────────────────────────────────────────────────┘
```

Client layer:
- [TypeScript](https://www.typescriptlang.org/)
- [TailwindCSS](https://tailwindcss.com/)
- [Shadcn/UI](https://www.shadcn-svelte.com/)
- [Astro](https://astro.build/)
- [Svelte](https://svelte.dev/)

API Layer:
- [Cloudflare workers](https://workers.cloudflare.com/)
- [HTMX](https://htmx.org/)
- [tRPC](https://trpc.io/)
- [GraphQL](https://graphql.org/)

Data Layer:
- Edge DB
- Object storage

### Summary

| Technology | Why | When to Skip |
|-----------|-----|--------------|
| Tailwind | Utility-first, performant, customizable | N/A |
| Shadcn | Accessible, unstyled, copy-paste | Simple UIs (just use Tailwind) |
| Astro | Minimal JS, islands, fast | N/A |
| Svelte | Small, fast, simple | No client-side reactivity needed |
| HTMX | Simple, performant, progressive | Data necessary before render |
| tRPC | Type-safe, great DX | Public APIs or can render server side |
| GraphQL | Flexible, self-documenting | Internal-only APIs |
| Workers | Edge, fast, scalable | No APIs |
| Edge DB | Low latency, global | No persistent data storage |

<h2 align="center">
  Decision Tree
</h2>

This tech stack is modular and does not need every technology to be sufficient for every web app. It is encouraged to only pick the relevant technology for your project. See the decision tree below and the following verbose description:

```
Is this a static page?
├─ Yes → Just Astro
└─ No → Where does the logic for behavior live?
   │
   ├─ Server-side only → Astro + HTMX (& Cloud worker(s))
   │
   ├─ Client-side only → Is server data required before rendering?
   │  ├─ No  → Astro + Svelte
   │  └─ Yes → AStro + Svelte + tRPC (& Cloud worker(s))
   │
   └─ Mix → Is server data required before rendering?
      ├─ No  → Astro + HTMX (& Cloud worker(s)) + Svelte
      └─ Yes → AStro + HTMX (& Cloud worker(s)) + Svelte + tRPC (& Cloud worker(s))

Worker language?
├─ tRPC? → TypeScript
└─ HTMX or GraphQL? → Rust

Is there a public API?
├─ Yes → GraphQL (& Cloud worker(s))
└─ No → No GraphQL
```

Astro on its own is fine for static site generation. However, when dynamic UI, reactivity, or fetching data from servers, other tools begin to become necessary.

For UIs where client-side logic and reactivity is needed to render, Svelte is the ideal solution. When server-side data is required for client-side logic *before* rendering, tRPC becomes relevant because it makes type-safe fetches without immediately rendering to HTML. However, if data isn't necessary for client-side logic, HTMX is ideal because it immediately renders the data and sends raw HTML instead of requiring the client to render the data.

For public APIs, both HTMX and tRPC are inadecate for both client specific, language lock-in, and security reasons. However, GraphQL shows superior over REST for distributing the exact data in single queries without over/under-fetching.

For tRPC, TypeScript is required for the cloud worker. However, for all other cloud workers, no langauge is required, so typically, Rust will be used.

<h2 align="center">
  Technology Overview
</h2>

## Client Layer

### TypeScript

- Catch errors at compile time, not runtime
- LSP autocompletion
- Self-documenting code through types
- Seamless integration with tRPC for end-to-end type safety
- No runtime overhead (types are erased at compile time)

The upfront cost of adding types is minimal, and the benefits compound as your codebase grows. If you're already comfortable with JavaScript, the learning curve is gentle.

---

### Astro

Static site generator with "islands architecture" that ships zero JavaScript and hydrates components only where needed.

- **Faster page loads**: Ships zero JavaScript by default. No virtual DOM or runtime. (ie - Next.js ships ~80KB of framework code minimum)
- **Better SEO**: Fully static HTML, no hydration delay. Search engines and site crawlers see all site content.
- **Simpler mental model**: No server/client components confusion. Static by default, interactive by exception.
- **Framework agnostic**: Use React, Svelte, Vue in the same project. Not locked in. Usually added for reactivity.
- **Islands architecture**: Add interactivity only where necessary. (ie - A blog post doesn't need React, but the comment form can be a Svelte island.)

**Performance comparison**:
| Framework                | Size                                   | Hydration |
|--------------------------|----------------------------------------|-----------|
| Traditional SPA (React): | 40KB framework overhead + Your code    | Hydration delay |
| Next.js:                 | 40KB framework overhead + Your code    | Partial hydration |
| Astro:                   | Your code only                         | Instant interaction |

*hydration is when client-side JavaScript takes over server-rendered HTML to make it interactive often causing slow loading*

We want to *minimize* hydration or totally avoid it when possible.

---

### Svelte

Component framework that compiles to vanilla JavaScript at build time. No virtual DOM.
Useful for interactive components that need reactivity (ie - forms, visualizations, real-time updates).

- **Smaller bundles**: No runtime framework. A Svelte component compiles to ~2KB, equivalent React is ~40KB (including React runtime).
- **Faster runtime**: No virtual DOM diffing. Updates are compiled at *build time* to direct DOM mutations.
- **Simpler syntax**: Less boilerplate than React hooks or Vue composition API.

Svelte: (6 lines)

```svelte
<script>
  let count = 0;
</script>
<button on:click={() => count++}>
  {count}
</button>
```

React: (8 lines + useState import)

```jsx
import { useState } from 'react';
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

- React/Vue: `state change → VDOM diff → DOM update` (runtime overhead)
- Svelte: `state change → DOM update` (compiled away *ahead-of-time*)

Adopt when you need a reactive component inside Astro islands and where bundle size and performance matters.

---

### TailwindCSS

Utility class CSS framework that provides all styling needs from layout to responsive design to animations.

- **Simpler mental model**: No naming conventions, no cascade conflicts, immediately clear.
- **Performance**: Purges unused styles at build time. Only ships CSS you actually use (~10KB typical production bundle vs 100KB+ for Bootstrap).
- **No context switching**: Write styles inline with markup. No jumping between files.
- **More customizable**: Full control over everything. Not locked into UI library's design opinions.
- **Consistent unit scale**: `p-4`, `m-2`, etc. instead of `padding: 17px` scattered everywhere.
- **Responsive design**: `md:flex lg:grid` makes media queries for differnt display sizes trivial.

**Utility classes over complex cascading**:

Traditional CSS is fragile and hard to change.

```css
/* Traditional CSS -  */
.card { padding: 1rem; }
.card-large { padding: 2rem; }
.card-large.featured { padding: 3rem; }
```

Which padding wins? It is non-obvious even for very small examples. And the cascading complexity grows exponentially with each line of styling.

On the contrary, Tailwind is explicit, predictable.

```html
<div class="p-4 lg:p-8">
```

It is obvious what element is getting styled and how.

Class names feel verbose initially, but productivity skyrockets once you internalize the system.

---

### Shadcn/UI (Headless Components)

Components without styling, built on Radix UI primitives, and styled with TailwindCSS.
Complex UI patterns (dropdowns, modals, date pickers) that need accessibility and behavior but without forced styling.

- **Tailwind-native**: Styled with utility classes, not fighting a separate styling system.
- **Accessible by default**: Built on Radix UI, which handles ARIA, keyboard nav, focus management.
  - Modals, dropdowns, and tooltips have subtle behaviors (focus trapping, escape handling, screen readers) that Radix solves.
  - Cross-browser quirks are handled.

For simple buttons and forms, just use Tailwind. For complex dropdowns, date pickers, command palettes—Shadcn saves weeks of accessibility work.

---

## API Layer

### Cloudflare Workers

Edge compute platform. Code runs in 300+ datacenters globally, close to users.
Serving APIs, rendering HTML, handling authentication—anything request/response.

- **Different auth**: Public API keys vs internal sessions
- **Different rate limits**: Stricter for public APIs
- **Different response formats**: HTML vs JSON
- **Independent scaling**: GraphQL might need more resources
- **Clear security boundaries**: Internal workers can have admin endpoints

---

### HTMX

Library that adds AJAX capabilities to HTML via attributes (`hx-get`, `hx-post`, etc.). Server returns HTML instead of JSON. Good for forms, CRUD operations, simple interactions where server has the logic.

- **Simpler**: No JSON ↔ HTML transformation. Server sends final markup.
- **Faster**: No double rendering (server generates data → client re-renders).
- **Less JavaScript**: ~14KB for HTMX vs ~40KB+ for React.
- **Progressive enhancement**: Works without JavaScript (with `<form>` fallback).
- **SPA-like experience**:
  - Preserve scroll position, focus
  - Animate transitions
  - Still get server rendering benefits

**Data flow comparison**:

Traditional SPA:
Client → Fetch JSON → Parse → Update state → Re-render → Update DOM

HTMX:
Client → Fetch HTML → Swap into DOM

---

### tRPC

End-to-end type-safe RPC for TypeScript. Call server functions as if they're local with full type inference. Good for fetching data when client-side logic is needed before rendering (filtering, transformations, complex state).

- **Type safety**: Changes to server types automatically break client code at compile time.
- **Better DX**: No manual API client code. `trpc.getUser.query('123')` just works.
- **Less boilerplate**: No OpenAPI specs, no manual type definitions.

```typescript
// Server
const appRouter = router({
  getUser: publicProcedure
    .input(z.string())
    .query(({ input }) => db.user.findById(input))
});

// Client - fully typed, autocomplete works
const user = await trpc.getUser.query('123');
//    ^? { id: string, name: string, email: string }
```

---

### GraphQL

Query language for APIs. Clients specify exactly what data they need. Good for public facing APIs.

- **Flexible queries**: Clients fetch exactly what they need. No over/under-fetching.
- **Single endpoint**: No `/users`, `/posts`, `/comments` proliferation.
- **Self-documenting**: Schema is contract. Tools auto-generate docs.
- **Versioning**: Adding fields doesn't break old clients. REST often needs `/v1`, `/v2`.
- **Strongly typed**: Schema validation built-in.

---

## Data Layer

### Edge Databases

Databases distributed globally with data replicated near users.

- **Lower read latency**: Data is near workers (1-5ms vs 50-100ms)
- **Consistency with edge workers**: Both code and data at edge
- **Built-in replication**: No manual multi-region setup
- **Cost-effective**: Many have generous free tiers

---

## Scenarios

### Summary

| Project Type | Astro | Svelte | HTMX | tRPC | GraphQL | Workers | Edge DB |
|--------------|-------|--------|------|------|---------|---------|---------|
| **Marketing Site** | ✅ Static shell | ⚠️ Contact form island | ⚠️ Newsletter signup | ❌ | ❌ | ⚠️ For form handling | ❌ |
| **Blog/Docs** | ✅ Core | ⚠️ Search widget | ⚠️ Comments | ❌ | ❌ | ⚠️ For comments | ⚠️ Store comments |
| **E-commerce** | ✅ Product pages | ✅ Cart/checkout | ✅ Product listings | ⚠️ Cart logic | ⚠️ If third-party | ✅ All APIs | ✅ Products/orders |
| **SaaS Dashboard** | ⚠️ Shell only | ✅ Charts/widgets | ⚠️ Simple forms | ✅ Complex data | ❌ | ✅ All APIs | ✅ User data |
| **Admin Panel** | ⚠️ Shell | ⚠️ Some widgets | ✅ CRUD forms | ⚠️ Reports | ❌ | ✅ All APIs | ✅ All data |
| **Real-time Collab** | ❌ | ✅ Entire UI | ❌ | ✅ + WebSockets | ❌ | ✅ All APIs | ✅ + Durable Objects |
| **API Platform** | ❌ | ❌ | ❌ | ❌ | ✅ Public API | ✅ GraphQL worker | ✅ All data |
| **Content + API** | ✅ Content | ⚠️ Embeds | ⚠️ Forms | ❌ | ✅ Public API | ✅ Separate workers | ✅ All data |

**Legend**: ✅ Primary use | ⚠️ Use selectively | ❌ Not applicable


### Scenario: Marketing Site with Contact Form

**Stack**: Astro + HTMX + Rust Worker + Edge DB

```
Static pages (Astro)
└─ Contact form (HTMX)
   └─ Rust worker validates & stores
      └─ Edge database
```

- Astro: Fast static page loads, perfect for marketing content
- HTMX: Simple form submission, server validates
- No Svelte needed: Form is simple
- No tRPC: No complex client logic
- Rust worker: Fast, handles form submission
- Edge DB: Store contact submissions

---

### Scenario: SaaS Dashboard

**Stack**: Astro + Svelte + tRPC + TypeScript Worker + Rust Worker + Edge DB

```
Static shell (Astro)
├─ Chart widgets (Svelte + tRPC)
├─ Data tables (Svelte + tRPC)
│  └─ TypeScript worker
│     └─ Edge database
└─ Simple settings forms (HTMX)
   └─ Rust worker
      └─ Edge database
```

- Astro: Static navigation, auth shell
- Svelte: Complex interactive widgets (charts, real-time data)
- tRPC: Client needs to transform data (filtering, aggregating) before rendering
- HTMX: Still use for simple forms (less overhead)
- TypeScript worker: Type safety across client/server boundary
- Edge DB: Fast queries for dashboard data

---

### Scenario: E-commerce Site

**Stack**: Astro + Svelte + HTMX + Rust Worker + Edge DB

```
Product pages (Astro SSG)
├─ Product search (HTMX)
├─ Cart widget (Svelte island)
└─ Checkout forms (HTMX)
   └─ Rust worker
      └─ Edge database (products, orders)
```

- Astro: Pre-render product pages (great SEO)
- HTMX: Search and simple forms (add to cart, checkout)
- Svelte: Shopping cart (complex state, needs reactivity)
- No tRPC: Most interactions are server-driven
- Rust worker: Fast, handles product queries and orders
- Edge DB: Product catalog and order history

---

### Scenario: Documentation Site with API

**Stack**: Astro + Svelte + GraphQL + Rust Worker + Edge DB

```
Docs (Astro SSG)
└─ Interactive code playground (Svelte island)

Separate API (GraphQL)
└─ Rust worker
   └─ Edge database
```

- Astro: Perfect for static documentation
- Svelte: Code playground needs interactivity
- GraphQL: Provide public API for docs content (third-party tools)
- Separate Rust worker: Public API isolated from docs site
- Edge DB: Store documentation content, make queryable via GraphQL

---

### Scenario: Admin Panel (CRUD Heavy)

**Stack**: Astro + HTMX + Rust Worker + Edge DB

```
Admin shell (Astro)
└─ All CRUD forms (HTMX)
   └─ Rust worker (handles all operations)
      └─ Edge database
```

- Astro: Simple navigation shell
- HTMX: Perfect for forms (create, edit, delete)
- No Svelte: Forms don't need complex client logic
- No tRPC: Server-driven is simpler
- Rust worker: Fast CRUD operations
- Edge DB: Store all admin data

---

### Scenario: Content Platform with Paid API

**Stack**: Astro + Svelte + HTMX + GraphQL + Rust Workers + Edge DB

```
Public site (Astro)
├─ Article pages (Astro SSG)
├─ Comments (HTMX)
├─ Embedded charts (Svelte islands)
└─ Internal tools (HTMX)
   └─ Rust worker
      └─ Edge database

Paid API (GraphQL)
└─ Rust worker (public GraphQL endpoint)
   └─ Edge database
```

- Astro: Static content for articles
- Svelte: Rich embeds (interactive charts, visualizations)
- HTMX: Comments and internal tools
- GraphQL: Paid public API (third-party access)
- Separate workers: Internal vs public isolation
- Both Rust workers: Internal and public workers
- Edge DB: Shared data layer
