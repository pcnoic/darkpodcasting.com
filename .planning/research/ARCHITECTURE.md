# Architecture Patterns

**Domain:** Atmospheric creative studio / podcast network static site
**Researched:** 2026-04-13
**Confidence:** HIGH (Astro/static-export patterns are mature and well-established; no WebSearch available, based on strong training knowledge through Aug 2025 + direct project analysis)

---

## Recommended Architecture

**Framework:** Astro (static output mode — `output: 'static'`)

The architecture is a **multi-page static site with a shared design system and per-show content islands**. Each page is a server-rendered Astro page at build time, producing a flat HTML tree. Interactive UI elements (animations, hover effects, audio player previews) are isolated React or Svelte islands that hydrate client-side only where needed.

```
darkpodcasting.com/
├── /                     → Homepage
├── /about                → Studio page
├── /shows/the-capital-archive
├── /shows/fabulas-de-machina
├── /shows/the-ashen-archives
└── /jobs                 → Jobs (existing, out-of-scope for rebrand)
```

There is no server, no database, and no runtime. The entire site is a folder of HTML/CSS/JS files served by nginx as-is.

---

## Component Boundaries

### Layer 1: Layout Shell (site-wide)

| Component | File | Responsibility | Communicates With |
|-----------|------|---------------|-------------------|
| `BaseLayout` | `src/layouts/BaseLayout.astro` | `<html>`, `<head>`, meta tags, font loading, global CSS reset | All pages |
| `Header` | `src/components/Header.astro` | Logo, nav links, mobile menu toggle | All page layouts |
| `Footer` | `src/components/Footer.astro` | Studio name, social links, copyright | All page layouts |

`BaseLayout` is the root — every page wraps itself in it. Header and Footer are composed inside it. No page renders without `BaseLayout`.

### Layer 2: Page Layouts (per page type)

| Component | File | Responsibility | Extends |
|-----------|------|---------------|---------|
| `HomeLayout` | `src/layouts/HomeLayout.astro` | Full-bleed hero, show grid structure | `BaseLayout` |
| `ShowLayout` | `src/layouts/ShowLayout.astro` | Show hero, episode list, lore section | `BaseLayout` |
| `StudioLayout` | `src/layouts/StudioLayout.astro` | About content, team layout | `BaseLayout` |

`ShowLayout` is the most important. It accepts a show's data object as a prop and renders the full show experience. All three show pages use it — no duplication.

### Layer 3: UI Components (atomic, reusable)

| Component | Responsibility | Used By |
|-----------|---------------|---------|
| `ShowCard` | Thumbnail + title + tagline card for show grid on homepage | `HomeLayout` |
| `EpisodeRow` | Single episode — title, number, duration, external link | `ShowLayout` |
| `AtmosphericHero` | Full-viewport section with background image, overlay, title reveal | `HomeLayout`, `ShowLayout` |
| `FogOverlay` | CSS/canvas atmospheric fog/grain effect | `AtmosphericHero`, optionally others |
| `SectionDivider` | Visual break between content sections (could be SVG, gradient, or rule) | All page layouts |
| `StudioMission` | Text block with the studio identity statement | `HomeLayout`, `StudioLayout` |

### Layer 4: Content (data, not components)

Show content lives in `src/content/shows/` as Markdown files with YAML frontmatter. Astro's **Content Collections** API handles schema validation and typed access at build time.

```
src/content/shows/
├── the-capital-archive.md
├── fabulas-de-machina.md
└── the-ashen-archives.md
```

Each file contains:
- Frontmatter: `title`, `slug`, `tagline`, `coverImage`, `accentColor`, `platforms` (Spotify/Apple/etc URLs), `status` (active/complete)
- Body: long-form lore/description text rendered as HTML in the show page

Episodes, if listed, are either embedded in frontmatter as a YAML array or stored as a separate `src/content/episodes/[show-slug]/` subfolder if volume justifies it. For three shows with modest episode counts, frontmatter arrays are sufficient and simpler.

---

## Data Flow

```
src/content/shows/*.md
        │
        ▼  (Astro Content Collections — build time)
src/pages/shows/[slug].astro
        │
        ▼  (getStaticPaths() enumerates slugs, getEntry() loads content)
ShowLayout.astro  ←──  show data object (typed, from collection schema)
        │
        ├──▶ AtmosphericHero  (receives: coverImage, title, tagline)
        ├──▶ EpisodeRow[]     (receives: episode array from frontmatter)
        └──▶ platform links   (receives: platforms map from frontmatter)
```

**Content update flow (new episode, show update):**
1. Edit the relevant `.md` file in `src/content/shows/`
2. Run `npm run build` — Astro regenerates static HTML
3. Push image to container registry, deploy updated pod to Kubernetes
4. Zero schema migrations. Zero API calls. Zero CMS login.

This is the right model for a small studio with infrequent content changes. It keeps all content in version control alongside the code.

**Homepage show grid data flow:**
```
src/content/shows/*.md
        │
        ▼  (getCollection('shows') — loads all three)
src/pages/index.astro
        │
        ▼
ShowCard[]   (one per show, ordered by frontmatter `order` field)
```

---

## Route Structure

| Route | File | Type | Notes |
|-------|------|------|-------|
| `/` | `src/pages/index.astro` | Static page | Homepage — hero + show grid + studio tag |
| `/about` | `src/pages/about.astro` | Static page | Studio identity, mission, team |
| `/shows/[slug]` | `src/pages/shows/[slug].astro` | Dynamic static (SSG) | `getStaticPaths()` generates one page per show |
| `/shows/the-capital-archive` | Generated | Static HTML | Resolved at build |
| `/shows/fabulas-de-machina` | Generated | Static HTML | Resolved at build |
| `/shows/the-ashen-archives` | Generated | Static HTML | Resolved at build |
| `/jobs` | `src/pages/jobs.astro` | Static page | Out of scope for rebrand; preserve as-is or port directly |

**nginx compatibility:** The existing `try_files $uri $uri/ /index.html` rule works for Astro's default output. Astro's `static` adapter emits `/shows/the-capital-archive/index.html`, so requests to `/shows/the-capital-archive` resolve correctly without any nginx changes.

**URL cleanliness:** Astro generates `trailingSlash: 'always'` or `'never'` — set to `'never'` in `astro.config.mjs` for consistency with the existing domain convention (no `.html` suffixes visible on current jobs page URL either).

---

## Full Project Layout

```
darkpodcasting.com/
├── public/
│   ├── images/
│   │   ├── shows/
│   │   │   ├── the-capital-archive/
│   │   │   │   ├── cover.jpg           (hero/card image)
│   │   │   │   └── og.jpg              (social share)
│   │   │   ├── fabulas-de-machina/
│   │   │   └── the-ashen-archives/
│   │   ├── studio/                     (team photos, studio imagery)
│   │   └── og-image.jpg               (site-level OG)
│   └── favicon.ico
├── src/
│   ├── content/
│   │   ├── config.ts                   (Astro collection schema — typed)
│   │   └── shows/
│   │       ├── the-capital-archive.md
│   │       ├── fabulas-de-machina.md
│   │       └── the-ashen-archives.md
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   ├── HomeLayout.astro
│   │   ├── ShowLayout.astro
│   │   └── StudioLayout.astro
│   ├── components/
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── ShowCard.astro
│   │   ├── EpisodeRow.astro
│   │   ├── AtmosphericHero.astro
│   │   ├── FogOverlay.astro
│   │   ├── SectionDivider.astro
│   │   └── StudioMission.astro
│   ├── pages/
│   │   ├── index.astro
│   │   ├── about.astro
│   │   ├── jobs.astro
│   │   └── shows/
│   │       └── [slug].astro
│   └── styles/
│       ├── global.css                  (CSS custom properties, resets)
│       ├── typography.css              (font scale, weights)
│       └── theme.css                  (dark palette, shadow tokens)
├── astro.config.mjs
├── tsconfig.json
├── package.json
├── Dockerfile                          (updated for new build output)
├── nginx.conf                          (unchanged)
└── k8s/                               (unchanged)
```

---

## Build Order (Phase Dependencies)

Components have a strict dependency order. Build bottom-up:

```
1. Theme tokens + global CSS  (no dependencies — everything else uses these)
       ↓
2. BaseLayout                 (depends on theme tokens, fonts)
       ↓
3. Header + Footer            (depend on BaseLayout)
       ↓
4. Content collection schema  (defines Show type — ShowLayout depends on it)
       ↓
5. AtmosphericHero + FogOverlay  (visual primitives — ShowLayout + HomeLayout use them)
       ↓
6. ShowCard + EpisodeRow      (atomic components — depend on Show type from schema)
       ↓
7. ShowLayout                 (assembles AtmosphericHero + EpisodeRow + platform links)
       ↓
8. HomeLayout                 (assembles AtmosphericHero + ShowCard[])
       ↓
9. Individual pages           (index, about, shows/[slug] — depend on their layouts)
       ↓
10. Docker build + K8s deploy (depends on completed static output)
```

Never build page-level work before the layout it depends on is stable. ShowLayout is the most reused and highest-risk component — stabilize it first by building it against one show (The Capital Archive), then apply to the other two.

---

## Patterns to Follow

### Pattern 1: Content Collections for Typed Show Data
**What:** Astro's `defineCollection` + Zod schema in `src/content/config.ts` defines the shape of every show's frontmatter. Build fails if a show file is missing required fields.
**When:** Always — eliminates runtime errors from missing content, makes show data self-documenting.

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const shows = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    slug: z.string(),
    tagline: z.string(),
    coverImage: z.string(),
    accentColor: z.string().regex(/^#[0-9a-fA-F]{6}$/),
    order: z.number(),
    platforms: z.object({
      spotify: z.string().url().optional(),
      apple: z.string().url().optional(),
      rss: z.string().url().optional(),
    }),
    episodes: z.array(z.object({
      number: z.number(),
      title: z.string(),
      duration: z.string().optional(),
      url: z.string().url().optional(),
    })).optional(),
  }),
});

export const collections = { shows };
```

### Pattern 2: Prop-Driven Layout Components
**What:** `ShowLayout` accepts a typed `show` prop and renders the entire show page from it. No prop drilling through 3 levels of children — layout receives the whole show object, distributes to its own children.
**When:** Always for the show pages — keeps `[slug].astro` a thin wrapper.

### Pattern 3: CSS Custom Properties for the Dark Theme
**What:** All atmospheric colors, shadow values, and blur amounts live as CSS custom properties on `:root` in `theme.css`. Components reference variables, never hardcoded hex values except in the theme file.
**When:** Always — makes future palette adjustments a single-file change.

```css
/* src/styles/theme.css */
:root {
  --color-void: #000000;
  --color-deep: #0a0a0a;
  --color-surface: #111111;
  --color-border: rgba(255,255,255,0.08);
  --color-text-primary: #f0ece4;
  --color-text-muted: #6b6460;
  --shadow-deep: 0 24px 80px rgba(0,0,0,0.9);
  --blur-fog: blur(80px);
}
```

### Pattern 4: Per-Show Accent Colors
**What:** Each show has an `accentColor` in its frontmatter. `ShowLayout` injects it as a CSS custom property scoped to that page (`style="--show-accent: {accentColor}"`). Components on that page use `var(--show-accent)` for glows, underlines, and highlights.
**When:** Always for show pages — gives each show visual identity without duplicating CSS.

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Hardcoding Show Content in Page Files
**What:** Writing show titles, taglines, and episode lists directly in `shows/the-capital-archive.astro` etc.
**Why bad:** Adding a fourth show requires creating a new page file and hunting down every place show data is repeated. Content and code become coupled.
**Instead:** All show content in `src/content/shows/*.md`; pages are generated from `[slug].astro`.

### Anti-Pattern 2: Skipping the Layout Abstraction
**What:** Building three separate show pages (no shared `ShowLayout`) because it feels faster initially.
**Why bad:** Any design change to the show page structure (rearranging hero/episodes/lore) must be made three times. Divergence accumulates.
**Instead:** Build `ShowLayout` once, battle-test it against one show, then apply to all three.

### Anti-Pattern 3: Putting Atmospheric Effects in Global CSS
**What:** Adding fog, grain, and glow animations at the `body` level so they "just work" everywhere.
**Why bad:** The homepage and show pages need different intensity levels. Global effects fight each other and create visual noise on transitional pages.
**Instead:** Atmospheric effects are components (`FogOverlay`, `AtmosphericHero`) placed explicitly in the layouts that need them, with intensity controlled via props.

### Anti-Pattern 4: React for Static Layout Components
**What:** Using React (or JSX) for components like `Header`, `ShowCard`, or `EpisodeRow` that have no interactive state.
**Why bad:** These ship unnecessary JavaScript to the browser. Astro renders them to zero-JS HTML at build time when written as `.astro` files.
**Instead:** Only use React (or Svelte) for genuinely interactive islands (e.g., a mobile nav menu toggle, or an embedded audio preview player). Everything structural is `.astro`.

### Anti-Pattern 5: Flat `public/images/` Directory
**What:** Dumping all images in one folder (`public/images/cover1.jpg`, `cover2.jpg`, etc.).
**Why bad:** Impossible to manage as the asset count grows; no clear ownership per show.
**Instead:** Namespace by show: `public/images/shows/the-capital-archive/cover.jpg`. Each show's images are co-located.

---

## Deployment Architecture

The deployment chain is unchanged from the current setup:

```
npm run build
    │  (Astro static export → dist/)
    ▼
docker build -t darkpodcasting-server .
    │  (copies dist/ into nginx image)
    ▼
Container Registry
    │
    ▼
Kubernetes Deployment (Rolling Update, maxUnavailable: 0)
    │
    ▼
nginx serves /usr/share/nginx/html/
    │
    ▼  (Ingress + TLS)
darkpodcasting.com
```

**Dockerfile change required:** The current Dockerfile copies `index.html`, `jobs.html`, and `images/` manually. After migration it should copy the entire `dist/` directory from the Astro build instead:

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Multi-stage build keeps the final image small (only nginx + static files, no Node.js runtime).

**nginx.conf:** No changes needed. `try_files $uri $uri/ /index.html` already handles Astro's directory-based URL structure (`/shows/the-capital-archive/` resolves to `dist/shows/the-capital-archive/index.html`).

**K8s manifests:** No changes needed. Same port, same health check endpoint, same rolling update strategy.

---

## Scalability Considerations

| Concern | At current scale (3 shows) | If shows grow (10+) |
|---------|---------------------------|---------------------|
| Content management | Frontmatter in `.md` files — trivial | Still fine; consider separating episodes into their own collection |
| Build time | Sub-30 seconds | Still fast; Astro's incremental build handles it |
| Image optimization | Manual (put files in `public/`) | Add Astro's `<Image>` component for automatic WebP + srcset |
| Animations | CSS + small JS islands | Same; no architectural change needed |
| Search / filtering | Not needed | Would require a client-side JSON index; Pagefind is the standard Astro solution |

---

## Sources

- Project analysis: `/Users/christos/src/github.com/pcnoic/darkpodcasting.com/.planning/PROJECT.md`
- Current site: `index.html`, `jobs.html`, `Dockerfile`, `nginx.conf`, `k8s/` manifests — direct inspection
- Astro documentation patterns (Content Collections, static output, `getStaticPaths`) — HIGH confidence from training data through Aug 2025; these are stable, production-grade APIs in Astro v4/v5
- Astro static adapter `output: 'static'` — standard pattern, no runtime changes needed to nginx or K8s
