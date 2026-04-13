# Technology Stack

**Project:** Dark Studios — Website Rebrand (darkpodcasting.com)
**Researched:** 2026-04-13
**Confidence:** MEDIUM-HIGH (framework and animation choices are verifiable from public docs; specific library versions flagged where training-data confidence only)

---

## Recommended Stack

### Core Framework

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Astro | 5.x (latest: ~5.6) | Static site framework, routing, component model | Purpose-built for content sites with full static output. Zero JS by default — only ships what you mark as interactive. File-based routing maps cleanly to the 5-page structure (home, 3 show pages, about). Native TypeScript, Vite-powered build. |
| Node.js | 20 LTS | Build runtime only (not deployed) | Astro's minimum supported runtime. Not present in final Docker image. |

**Why Astro over Next.js:** Next.js 15 is optimized for hybrid SSR, React Server Components, and App Router — none of which apply to a fully static, no-backend, no-CMS site. Choosing Next.js introduces mandatory React runtime on every page, RSC plumbing, and a build output that requires careful configuration to generate true static HTML. Astro's `output: 'static'` mode (the default) produces a flat directory of `.html` files that drops directly into the existing nginx COPY step. The cognitive overhead of Next.js for 5 static pages is unjustified.

**Why Astro over Vite + React SPA:** A single-page app creates SEO fragility (content gated behind JS execution), adds client-side routing complexity, and ships an entire React bundle to every visitor before any content renders. Astro's multi-page static output is a strict improvement for a studio marketing site.

**Why not Gatsby:** Gatsby 5 is in maintenance mode as of 2023. Netlify (the acquirer) has signaled no active investment. Do not use.

**Why not SvelteKit or Nuxt:** Both are excellent, but the team expressed React familiarity in the PROJECT.md ("React or Astro TBD"). Astro supports React components as islands if needed, giving a migration path. Switching to Svelte or Vue adds language overhead for no gain on this project scope.

---

### Styling

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Tailwind CSS | 4.x (v4.0 released Jan 2025) | Utility-first CSS | Tailwind v4 is CSS-first — configured via `@import "tailwindcss"` in a CSS file, no `tailwind.config.js` required for standard usage. Works natively with Astro via the `@astrojs/tailwind` integration (or direct PostCSS in v4). For dark atmospheric design, utility classes make iteration fast: `bg-black/90 backdrop-blur-lg shadow-[0_0_80px_rgba(0,0,0,0.9)]`. Scoped styles in Astro components complement Tailwind for component-specific overrides. |

**Why not CSS Modules only:** CSS Modules are fine but slow for atmospheric design iteration — every fog gradient, shadow-spread, and opacity variant requires a named class. Tailwind's utilities are faster to compose and easier to adjust in review. Astro's scoped `<style>` blocks remain available for truly component-specific rules.

**Why not styled-components / Emotion:** These are React-specific CSS-in-JS libraries. They add a JS runtime cost, don't work in Astro's non-island `.astro` components, and complicate static generation. Wrong tool for this stack.

---

### Animation

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| GSAP (GreenSock) | 3.x (latest: ~3.12) | Scroll-driven cinematic animations, entrance reveals, parallax | GSAP is framework-agnostic. It targets DOM elements directly — no React lifecycle dependency, no virtual DOM reconciliation. Works identically in Astro's `<script>` blocks. ScrollTrigger (bundled free) is the industry standard for scrubbed timeline animations tied to scroll position: parallax headers, fog reveal on show pages, title scrub effects. GSAP handles all timing edge cases (mobile scroll inertia, resize recalculation) that CSS animations and simple JS do not. |
| CSS animations + `@keyframes` | Native | Looping subtle effects (fog drift, particle float, ambient pulse) | For purely looping, non-interactive effects, CSS `@keyframes` with `animation` properties are more performant than JS-driven loops (run on compositor thread). Reserve for "always on" effects that never need to be scrubbed or programmatically controlled. |

**Why GSAP over Framer Motion:** Framer Motion is coupled to React's render lifecycle. In Astro, using Framer Motion requires wrapping every animated element in a React island (`client:load` or `client:visible`), which ships a React runtime just to animate a `<h1>`. GSAP runs in a plain `<script>` tag, has access to any DOM element regardless of component origin, and its ScrollTrigger plugin is significantly more mature for cinematic scroll-scrubbing than Framer Motion's `useScroll` / `useTransform` hooks. For a studio site with atmospheric scroll reveals, GSAP is the correct choice.

**Why GSAP over Motion (formerly Framer Motion standalone):** Motion v11+ as a framework-agnostic library is promising but has a smaller documentation surface, fewer production examples for the specific cinematic patterns (parallax image scrub, staggered text reveal, fog overlay) that Dark Studios needs. GSAP has years of studio/portfolio site examples and a robust community. Confidence in Motion's standalone API stability is lower.

**Why not Three.js in Phase 1:** Three.js WebGL scenes (fog, particles, atmospheric depth) are a strong differentiator but carry real implementation cost. Treat as a Phase 2+ enhancement. Astro's island model makes adding a Three.js scene to the homepage hero trivially isolated once the base site is built.

---

### Fonts

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Fontsource | npm packages (per-font) | Self-hosted web fonts | Fontsource packages Google Fonts as npm modules. Self-hosting avoids the Google Fonts CDN DNS lookup, eliminates Google tracking concerns, and enables `font-display: swap` tuning. Fonts become build-time dependencies and are copied to the static output — no runtime CDN dependency. |

**Recommended typeface direction (not a code dependency — a design decision):**
- Drop Jersey 10 Charted and JetBrains Mono as primary faces. They are terminal aesthetic — exactly what the rebrand is moving away from.
- Cinzel Decorative (serif, Roman, gothic weight) — for display headings and show titles. Evokes antiquity and dread. Available on Fontsource.
- Inter or DM Sans — for body copy and UI labels. Clean, readable on dark backgrounds at small sizes. Available on Fontsource.
- JetBrains Mono — retain only for episode timestamps, code-like metadata if used. Do not use as a primary face.

Confidence on typeface selection: LOW (aesthetic judgment, not technical fact). Verify against brand direction.

---

### Build & Deployment

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Multi-stage Dockerfile | — | Build inside Docker, serve with nginx | The current Dockerfile `COPY index.html` approach must change. Stage 1: `node:20-alpine` runs `astro build` to produce the `dist/` directory. Stage 2: `nginx:alpine` copies `dist/` to `/usr/share/nginx/html/`. Final image contains only nginx + static files — no Node.js runtime in production. Same image size profile as current. |
| Vite | 6.x (bundled with Astro) | Build tooling | Astro uses Vite internally. No separate Vite configuration needed for standard use. |

**Dockerfile change summary:**

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

The nginx.conf `try_files $uri $uri/ /index.html` directive is already correct for Astro's static output (Astro generates individual `index.html` per route, so the fallback is rarely exercised but harmless).

The GitHub Actions workflow path triggers (`index.html`, `jobs.html`) must be updated to include `src/**` and `public/**` when the Astro project structure is introduced.

---

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Framework | Astro 5 | Next.js 15 | Next.js is SSR/hybrid-first; static mode requires explicit config; adds React RSC runtime complexity for no benefit on a 5-page static site |
| Framework | Astro 5 | Vite + React SPA | SPA adds JS-gated content (SEO risk), client-side routing, full React bundle on first load |
| Framework | Astro 5 | Gatsby 5 | Maintenance mode; ecosystem has moved on |
| Animation | GSAP | Framer Motion | React lifecycle coupling; requires React islands for DOM elements; ScrollTrigger is more mature than Framer's scroll hooks for cinematic scrubbing |
| Animation | GSAP | Motion (standalone) | Smaller community, less documentation for cinematic patterns, uncertain API stability trajectory |
| Styling | Tailwind v4 | CSS Modules only | Slower iteration for atmospheric one-off utilities; Tailwind still emits standard CSS — no runtime overhead |
| Styling | Tailwind v4 | styled-components | React-specific, adds JS runtime, incompatible with Astro non-island components |
| Fonts | Fontsource | Google Fonts CDN | Adds DNS lookup, Google tracking, runtime CDN dependency — eliminated by self-hosting |

---

## Installation

```bash
# Scaffold Astro project
npm create astro@latest -- --template minimal

# Astro integrations
npx astro add tailwind

# Animation
npm install gsap

# Fonts (example — confirm typefaces in design phase)
npm install @fontsource/cinzel-decorative
npm install @fontsource/inter
npm install @fontsource/jetbrains-mono

# TypeScript (included with Astro by default)
# No additional install needed
```

---

## Confidence Assessment

| Recommendation | Confidence | Basis |
|----------------|------------|-------|
| Astro as framework | HIGH | Astro's static output model and content-site focus are well-established and stable; static-first is the documented default |
| Astro version ~5.6 | MEDIUM | Based on training data (Astro 5 released ~Dec 2024); verify exact latest via `npm view astro version` before scaffolding |
| GSAP for animation | HIGH | Framework-agnostic, ScrollTrigger is industry standard, works trivially in Astro `<script>` tags |
| GSAP version ~3.12 | MEDIUM | Training data; verify via `npm view gsap version` |
| Tailwind v4 | HIGH | v4 released January 2025, CSS-first config, confirmed Astro integration path exists |
| Multi-stage Dockerfile | HIGH | Standard pattern, no novel technology, fully supported |
| Fontsource | HIGH | Standard self-hosting pattern, well-maintained, direct npm install |
| Typeface direction | LOW | Aesthetic judgment, not verified against brand guidelines |
| Three.js deferral | MEDIUM | Judgment call based on implementation cost vs phase scope |

---

## Sources

- Astro documentation (astro.build/docs) — static output, islands architecture, Tailwind integration
- GSAP documentation (gsap.com/docs) — ScrollTrigger, free tier capabilities
- Tailwind CSS v4 announcement (tailwindcss.com/blog/tailwindcss-v4) — CSS-first config, January 2025 release
- Fontsource (fontsource.org) — npm-installable Google Fonts
- Project files: `Dockerfile`, `nginx.conf`, `k8s/base/deployment.yaml`, `.github/workflows/deploy.yml` — confirms static nginx deployment, Docker multi-stage feasibility
- `index.html` — confirms current terminal aesthetic (green #00ff00, JetBrains Mono) being replaced

Note: WebSearch and WebFetch were not available in this research session. Version numbers marked MEDIUM confidence should be verified against npm registry before scaffolding. Core architectural choices (Astro, GSAP, Tailwind) are HIGH confidence from training data and internal consistency.
