---
name: astro-optimize
description: >
  Expert-level Astro framework optimization covering islands architecture, partial hydration,
  content collections, View Transitions, image optimization, bundle splitting, SSG/SSR/hybrid
  output modes, middleware, and deployment adapters. Use this skill whenever the user mentions
  Astro, .astro files, astro.config.mjs, islands, partial hydration, content collections,
  Astro integrations, View Transitions, or asks about performance/optimization/deployment
  in an Astro project — even if the question seems simple. Also trigger for questions about
  deploying Astro to Vercel, Netlify, Cloudflare, or Node.js adapters. Trigger for ANY
  Astro-related question regardless of complexity.
---

# Astro Optimize

## 🎯 Goal
Maximize performance in Astro projects: ship minimal JavaScript, render as much as possible on the server, and hydrate only what truly needs to be interactive.

---

## 🔒 Hard Rules (Never Skip)

- Default to **zero JavaScript** — no `client:*` directive unless the component is interactive
- Never use `client:load` when `client:idle` or `client:visible` works
- Never hydrate static/presentational components
- Never use raw `<img>` for local assets — always `<Image />` or `<Picture />`
- Always define `width`, `height`, and `alt` on every image
- Always set `site` in `astro.config.mjs` — required for canonical URLs and sitemap
- Use **Content Collections** for all structured content — never `import.meta.glob()` raw markdown

---

## 🧠 Core Philosophy

- **Static first**: if a page doesn't need dynamic data, it's `output: 'static'`
- **Islands over SPAs**: hydrate individual components, not entire layouts
- **No framework wrapping**: don't wrap full pages in React/Vue — use Astro layouts
- **Build-time over runtime**: compute everything you can at build time

---

## 🏝️ Islands Architecture — Partial Hydration

### Client Directive Decision Table

| Directive | When to Use |
|---|---|
| *(none)* | Static/presentational — icons, headings, cards, text |
| `client:idle` | Interactive but not urgent — likes, share buttons, comments |
| `client:visible` | Heavy widgets, charts, embeds below the fold |
| `client:load` | Auth state, shopping cart, anything needing instant reactivity |
| `client:media` | Responsive patterns — mobile nav, drawers |

```astro
<!-- ✅ Correct usage -->
<StaticCard />                          <!-- 0 KB JS -->
<ShareButton client:idle />             <!-- non-urgent interaction -->
<HeavyChart client:visible />           <!-- below fold, heavy -->
<HeavyChart client:visible={{ rootMargin: "200px" }} />  <!-- preload slightly early -->
<Cart client:load />                    <!-- needs instant state -->
<MobileNav client:media="(max-width: 768px)" />

<!-- 🚫 Anti-pattern -->
<StaticHero client:load />              <!-- never hydrate static content -->
<Layout client:load>...</Layout>        <!-- never hydrate entire layouts -->
```

---

## 🖼️ Images

Always use `<Image />` or `<Picture />` from `astro:assets`.

```astro
---
import { Image, Picture } from 'astro:assets';
import heroImg from '../assets/hero.jpg';
---

<!-- LCP image: eager + high fetchpriority -->
<Image
  src={heroImg}
  alt="Descriptive alt text"
  width={1200}
  height={630}
  quality={80}
  loading="eager"
  fetchpriority="high"
/>

<!-- Responsive + multi-format (best compression: 30–50% smaller) -->
<Picture
  src={heroImg}
  formats={['avif', 'webp']}
  widths={[400, 800, 1200]}
  sizes="(max-width: 768px) 100vw, 50vw"
  alt="Hero image"
/>
```

### Image Checklist
- [ ] LCP image: `loading="eager"` + `fetchpriority="high"`
- [ ] Below fold: `loading="lazy"` (Astro default — no need to specify)
- [ ] Always set `width` + `height` — prevents CLS
- [ ] Use `formats={['avif', 'webp']}` for best compression
- [ ] Add `sizes` for responsive images
- [ ] `alt=""` for decorative images, descriptive text for content

---

## 🔤 Fonts

**Best**: self-host via `fontsource` — zero external request, full control.

```bash
pnpm add @fontsource-variable/inter
```
```astro
---
import '@fontsource-variable/inter/wght.css';
---
```

**If using Google Fonts**: load non-blocking to avoid render delay.

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap"
  media="print"
  onload="this.media='all'" />
```

---

## 📦 astro.config.mjs — Performance Baseline

```js
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://yoursite.com',   // REQUIRED — canonical URLs + sitemap

  output: 'static',               // 'static' | 'server' | 'hybrid'

  prefetch: {
    prefetchAll: false,           // Never prefetch everything — wastes bandwidth
    defaultStrategy: 'viewport', // 'hover' | 'tap' | 'viewport' | 'load'
  },

  build: {
    inlineStylesheets: 'auto',   // Inline small CSS, link large files
    concurrency: 4,              // Parallel page rendering
  },

  image: {
    domains: ['cdn.example.com'],
    remotePatterns: [{ protocol: 'https', hostname: '**.cloudinary.com' }],
  },

  vite: {
    build: {
      cssMinify: 'lightningcss',
      rollupOptions: {
        output: {
          // Split heavy vendor libs into separate chunks
          manualChunks: {
            'chart': ['chart.js'],
            'editor': ['@codemirror/state', '@codemirror/view'],
          },
        },
      },
    },
    css: { transformer: 'lightningcss' },
  },

  integrations: [sitemap()],
});
```

---

## 🗂️ Output Mode — When to Use Each

| Scenario | Mode | Adapter |
|---|---|---|
| Blog, docs, marketing site | `static` | None |
| Auth, user data, API routes | `server` | `@astrojs/vercel` / `@astrojs/node` |
| Mostly static + a few dynamic routes | `hybrid` | Any |
| Ecommerce, dashboard | `server` or `hybrid` | Vercel / Cloudflare |

```astro
---
// In hybrid mode — opt individual pages in/out of SSR:
export const prerender = false;   // This page = SSR
export const prerender = true;    // This page = static (even in 'server' mode)
---
```

---

## 📚 Content Collections

```ts
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content', // 'content' = md/mdx | 'data' = json/yaml
  schema: ({ image }) => z.object({
    title: z.string().max(60),
    description: z.string().max(160),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    heroImage: image().optional(),  // image() enables optimization
    draft: z.boolean().default(false),
    tags: z.array(z.string()).default([]),
  }),
});

export const collections = { blog };
```

```ts
// Filter drafts in production, sort by date
const posts = await getCollection('blog', ({ data }) =>
  import.meta.env.PROD ? !data.draft : true
);
const sorted = posts.sort((a, b) =>
  b.data.pubDate.valueOf() - a.data.pubDate.valueOf()
);
```

---

## 🎬 View Transitions

```astro
---
import { ViewTransitions } from 'astro:transitions';
---
<head>
  <ViewTransitions />
</head>
```

```astro
<!-- Morph matched elements across pages -->
<h1 transition:name="hero-title">{post.title}</h1>
<img transition:name={`post-img-${post.slug}`} />

<!-- Built-in animations -->
<div transition:animate="slide" />
<div transition:animate="fade" />

<!-- Persist interactive state (great for audio/video) -->
<AudioPlayer transition:persist />
```

```ts
// Re-init third-party scripts after each navigation
document.addEventListener('astro:page-load', () => {
  initAnalytics();
});
```

---

## ⚡ Performance Targets

| Metric | Target | Key Fix |
|---|---|---|
| LCP | < 2.5s | Eager + fetchpriority on hero image |
| CLS | < 0.1 | Always set width/height on images |
| INP | < 200ms | Use `client:idle` over `client:load` |
| JS Bundle | Minimal | No `client:load` on static content |

---

## 🚫 Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| `client:load` on every component | Use `client:idle` or `client:visible` |
| `import.meta.glob('**/*.md')` | Use Content Collections |
| Raw `<img>` for local assets | Use `<Image />` from `astro:assets` |
| No `sizes` on responsive images | Add `sizes` with breakpoints |
| Google Fonts blocking render | Use fontsource or `media="print"` trick |
| Missing `site` in config | Set `site` — required for sitemap + canonical |
| Fetching data in every component | Fetch at page level, pass as props |
| Large scripts loaded synchronously | Defer with `async`/`defer` or use Partytown |
| Wrapping full pages in React | Use Astro layouts, hydrate islands only |

---

## ✅ Definition of Done

A page is **NOT complete** unless:
- [ ] It ships zero unnecessary JavaScript
- [ ] All images use `<Image />` / `<Picture />` with `width`, `height`, `alt`
- [ ] No `client:load` without explicit justification
- [ ] LCP image has `loading="eager"` + `fetchpriority="high"`
- [ ] `site` is set in `astro.config.mjs`
- [ ] Fonts are self-hosted or loaded non-blocking
- [ ] Content Collections used for all structured content
