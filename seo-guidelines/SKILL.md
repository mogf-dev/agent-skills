---
name: seo-guidelines
description: >
  World-class SEO implementation guide covering meta tags, Open Graph, Twitter Cards,
  JSON-LD structured data, canonical URLs, sitemap.xml, robots.txt, Core Web Vitals,
  semantic HTML, hreflang for i18n, and framework-specific patterns for Astro and Next.js.
  Use this skill whenever the user asks about SEO, search engine optimization, meta tags,
  Open Graph, structured data, JSON-LD, Schema.org, sitemap, robots.txt, Core Web Vitals,
  PageSpeed, Lighthouse scores, Google Search Console, indexability, canonical tags,
  hreflang, or improving search rankings — even for simple SEO questions. Also trigger
  when the user mentions "rank higher", "Google", "search traffic", or "discoverability".
---

# SEO Guidelines

## 🎯 Goal
Ensure every page is optimized for search engines, social sharing, and discoverability. No page should ship without proper SEO fundamentals.

---

## 🔒 Hard Rules (Never Skip)

Every page **MUST** have:
- `<title>` — ≤ 60 characters, keyword-focused, unique per page
- `<meta name="description">` — ≤ 160 characters, unique per page
- `<link rel="canonical">` — self-referencing on every page
- Open Graph tags: `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- Twitter Card tags: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`

Content rules:
- Exactly **ONE `<h1>`** per page
- Proper heading hierarchy: `h1 → h2 → h3` — never skip levels
- All images **MUST** have meaningful `alt` text (`alt=""` only for decorative)
- Use descriptive anchor text — never "click here" or "read more"
- Every page links to at least 2–3 relevant internal pages

---

## 🧩 Reusable SEO Component (Astro)

```astro
---
// src/components/SEO.astro
interface Props {
  title: string;
  description: string;
  image?: string;
  canonicalURL?: string;
  type?: 'website' | 'article' | 'product';
  publishDate?: Date;
  modifiedDate?: Date;
  author?: string;
  noindex?: boolean;
}

const {
  title,
  description,
  image = '/og-default.jpg',
  canonicalURL = Astro.url.href,
  type = 'website',
  publishDate,
  modifiedDate,
  author,
  noindex = false,
} = Astro.props;

const siteUrl = Astro.site ?? 'https://yoursite.com';
const absoluteImage = image.startsWith('http') ? image : new URL(image, siteUrl).href;
const formattedTitle = `${title} | Your Site Name`;
---

<!-- Primary -->
<title>{formattedTitle}</title>
<meta name="description" content={description} />
<link rel="canonical" href={canonicalURL} />
{noindex && <meta name="robots" content="noindex, nofollow" />}

<!-- Open Graph -->
<meta property="og:type" content={type} />
<meta property="og:url" content={canonicalURL} />
<meta property="og:title" content={formattedTitle} />
<meta property="og:description" content={description} />
<meta property="og:image" content={absoluteImage} />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:site_name" content="Your Site Name" />
<meta property="og:locale" content="en_US" />

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:site" content="@yourhandle" />
<meta name="twitter:title" content={formattedTitle} />
<meta name="twitter:description" content={description} />
<meta name="twitter:image" content={absoluteImage} />

<!-- Article-specific -->
{type === 'article' && publishDate && (
  <meta property="article:published_time" content={publishDate.toISOString()} />
)}
{type === 'article' && modifiedDate && (
  <meta property="article:modified_time" content={modifiedDate.toISOString()} />
)}
{type === 'article' && author && (
  <meta property="article:author" content={author} />
)}
```

```astro
<!-- Usage in any page -->
<head>
  <SEO
    title="How to Build Fast Websites"
    description="Learn proven techniques to build websites that load in under 1 second."
    type="article"
    publishDate={new Date('2024-01-15')}
    image="/blog/fast-websites-og.jpg"
  />
</head>
```

---

## 🧠 Structured Data (JSON-LD)

Use JSON-LD when applicable. Always validate with [Rich Results Test](https://search.google.com/test/rich-results).

### Article / BlogPosting
```ts
const articleSchema = {
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": post.data.title,
  "description": post.data.description,
  "image": absoluteImageUrl,
  "datePublished": post.data.pubDate.toISOString(),
  "dateModified": (post.data.updatedDate ?? post.data.pubDate).toISOString(),
  "author": { "@type": "Person", "name": post.data.author, "url": `${siteUrl}/about` },
  "publisher": {
    "@type": "Organization",
    "name": "Your Site Name",
    "logo": { "@type": "ImageObject", "url": `${siteUrl}/logo.png` }
  },
  "mainEntityOfPage": { "@type": "WebPage", "@id": canonicalURL }
};
```

### Organization (Homepage)
```ts
const orgSchema = {
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Your Company",
  "url": siteUrl,
  "logo": `${siteUrl}/logo.png`,
  "sameAs": [
    "https://twitter.com/yourhandle",
    "https://github.com/yourorg",
    "https://linkedin.com/company/yourco"
  ],
};
```

### WebSite (Sitelinks search box)
```ts
const websiteSchema = {
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Your Site Name",
  "url": siteUrl,
  "potentialAction": {
    "@type": "SearchAction",
    "target": { "@type": "EntryPoint", "urlTemplate": `${siteUrl}/search?q={search_term_string}` },
    "query-input": "required name=search_term_string"
  }
};
```

### FAQ (Expandable answers in SERPs)
```ts
const faqSchema = {
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": faqs.map(({ question, answer }) => ({
    "@type": "Question",
    "name": question,
    "acceptedAnswer": { "@type": "Answer", "text": answer }
  }))
};
```

### Product
```ts
const productSchema = {
  "@context": "https://schema.org",
  "@type": "Product",
  "name": product.name,
  "description": product.description,
  "image": product.images,
  "offers": {
    "@type": "Offer",
    "price": product.price,
    "priceCurrency": "USD",
    "availability": product.inStock
      ? "https://schema.org/InStock"
      : "https://schema.org/OutOfStock",
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": product.rating,
    "reviewCount": product.reviewCount
  }
};
```

```astro
<!-- Inject any schema -->
<script type="application/ld+json" set:html={JSON.stringify(schema)} />
```

---

## 🗺️ Indexing

### sitemap.xml
```js
// astro.config.mjs
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://yoursite.com',   // REQUIRED
  integrations: [
    sitemap({
      filter: (page) =>
        !page.includes('/draft/') &&
        !page.includes('/admin/') &&
        !page.includes('/404'),
      changefreq: 'weekly',
      priority: 0.7,
      serialize(item) {
        if (item.url.includes('/blog/')) {
          return { ...item, priority: 0.9, changefreq: 'monthly' };
        }
        return item;
      },
    }),
  ],
});
```

### robots.txt
```txt
# public/robots.txt
User-agent: *
Allow: /

Disallow: /admin/
Disallow: /private/
Disallow: /api/
Disallow: /*.json$

Sitemap: https://yoursite.com/sitemap-index.xml
```

---

## 🔗 URL & Canonical Rules

- Lowercase URLs: `/blog/my-post` not `/Blog/My-Post`
- Hyphens as separators: `/my-post` not `/my_post`
- Consistent trailing slashes — pick one, never mix
- Self-referencing canonical on **every** page, including paginated
- 301 (not 302) for all permanent URL changes

```astro
<!-- Pagination signals -->
{page.url.prev && <link rel="prev" href={page.url.prev} />}
{page.url.next && <link rel="next" href={page.url.next} />}
```

---

## ⚡ Performance & Core Web Vitals

Avoid blocking scripts in `<head>`. Prefer static content over client-rendered content.

| Metric | Target | Key Fix |
|---|---|---|
| LCP | < 2.5s | `loading="eager"` + `fetchpriority="high"` on hero image |
| CLS | < 0.1 | Always set `width`/`height` on images, reserve space for embeds |
| INP | < 200ms | Minimize JS, debounce handlers, avoid `client:load` |
| FCP | < 1.8s | Inline critical CSS, preconnect to critical origins |

```astro
<!-- Preload LCP image -->
<link rel="preload" as="image" href="/hero.avif"
  imagesrcset="/hero-400.avif 400w, /hero.avif 1200w"
  imagesizes="100vw" />

<!-- Preconnect to critical third parties -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="dns-prefetch" href="https://analytics.example.com" />
```

---

## 🌍 Internationalization (hreflang)

```astro
---
const alternateLocales = [
  { lang: 'en', url: `${siteUrl}/en${path}` },
  { lang: 'es', url: `${siteUrl}/es${path}` },
  { lang: 'x-default', url: `${siteUrl}/en${path}` },  // Fallback
];
---
{alternateLocales.map(({ lang, url }) => (
  <link rel="alternate" hreflang={lang} href={url} />
))}
```

---

## 🚫 Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Missing or duplicate `<title>` / `<meta description>` | Unique tags on every page |
| Multiple `<h1>` per page | Exactly one, matches primary keyword |
| Empty or generic descriptions | Unique, 150–160 chars, compelling copy |
| Images without `alt` text | Always add; `alt=""` only for decorative |
| "Click here" anchor text | Descriptive text explaining the destination |
| `noindex` left on production pages | Audit robots directives before going live |
| Missing canonical on paginated pages | Add self-referencing canonical always |
| Client-rendered content only | Prefer SSR/SSG — crawlers struggle with CSR |
| Blocking scripts in `<head>` | Move to end of body or use `defer`/`async` |
| No sitemap submitted | Generate + submit to Google Search Console |

---

## ✅ Definition of Done

A page is **NOT complete** unless:
- [ ] Unique `<title>` (≤ 60 chars) and `<meta description>` (≤ 160 chars)
- [ ] Self-referencing `<link rel="canonical">`
- [ ] Full Open Graph + Twitter Card tags
- [ ] Exactly one `<h1>`, proper heading hierarchy
- [ ] All images have meaningful `alt` text
- [ ] JSON-LD structured data where applicable (Article, FAQ, Product, Org)
- [ ] Page appears in sitemap (unless intentionally excluded)
- [ ] robots.txt allows the page
- [ ] LCP < 2.5s, CLS < 0.1 (verified in PageSpeed Insights)
- [ ] Validated with [Google Rich Results Test](https://search.google.com/test/rich-results)

---

## 🛠️ Tools
- **Google Search Console** — indexing status, Core Web Vitals, manual actions
- **PageSpeed Insights** (`pagespeed.web.dev`) — field + lab data
- **Rich Results Test** (`search.google.com/test/rich-results`) — validate JSON-LD
- **Schema Validator** (`validator.schema.org`) — catch schema errors
- **Screaming Frog** (free ≤ 500 pages) — full technical crawl audit
