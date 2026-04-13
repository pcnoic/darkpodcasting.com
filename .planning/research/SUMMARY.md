# Project Research Summary

**Project:** Dark Studios — Website Rebrand (darkpodcasting.com)
**Domain:** Atmospheric creative studio / podcast network static site
**Researched:** 2026-04-13
**Confidence:** HIGH

## Executive Summary

Dark Studios is a cinematic audio drama brand replacing a terminal-aesthetic single-page site with a multi-page atmospheric studio presence. The research consensus is clear: this is a content-first static site with atmospheric visual ambition. Astro 5 is the unambiguous framework choice — it produces flat HTML at build time with zero JS shipped by default, maps directly to the five-page URL structure, and runs GSAP animations in plain `<script>` tags without React lifecycle dependency. The existing Docker/Kubernetes/nginx infrastructure requires only one meaningful change: the Dockerfile becomes a two-stage build that runs `astro build` and copies `dist/` into the nginx image rather than copying individual HTML files manually.

The recommended approach is to build the design system first (theme tokens, typography, contrast values), then the shared `ShowLayout` component wired to Astro Content Collections, then all three show pages, then homepage and About. Each show page follows an identical structure — atmospheric hero, show-specific accent color, lore section, episode list, listen links — driven entirely from a typed Markdown frontmatter schema. This architecture means adding a new show costs one `.md` file, not a new page file. GSAP ScrollTrigger handles cinematic scroll reveals but must not be introduced until layout is stable; premature animation implementation is the single most common wasted-effort failure mode on this type of project.

The primary risks are performance and accessibility: atmospheric dark sites routinely fail on both. The combination of unoptimized hero images (common), low contrast text on near-black backgrounds (common), and carry-over CSS from the terminal site — especially `overflow: hidden` on `body` — can each independently destroy the experience. All three must be addressed in Phase 1 before any page content is built. The good news: this project has no authentication, no API, no CMS, no state management, and no real-time data. The scope is tight and the architecture is proven.

---

## Key Findings

### Recommended Stack

Astro 5 (static output mode) is purpose-built for this use case. It ships zero JavaScript by default, supports file-based routing that maps directly to the five-page structure, and integrates with GSAP via plain `<script>` tags — no React runtime required. Tailwind CSS v4 (CSS-first configuration, no `tailwind.config.js`) handles utility composition for the dark palette. GSAP 3.x with ScrollTrigger handles cinematic scroll-driven reveals. Fontsource provides self-hosted web fonts as npm packages, eliminating the Google Fonts CDN dependency. The Dockerfile becomes a standard two-stage build: `node:20-alpine` runs `astro build`, `nginx:alpine` serves `dist/`. K8s manifests require no changes.

**Core technologies:**
- **Astro 5** (static site framework) — purpose-built for content sites, zero JS by default, file-based routing, TypeScript native
- **GSAP 3.x + ScrollTrigger** (animation) — framework-agnostic, runs in plain `<script>` tags, industry standard for cinematic scroll scrubbing
- **Tailwind CSS v4** (styling) — CSS-first utility composition; fast iteration for atmospheric one-off shadows and overlays
- **Fontsource** (typography) — self-hosted fonts as npm packages; `font-display` control; no CDN dependency
- **Multi-stage Dockerfile** (deployment) — node:20-alpine build stage, nginx:alpine serve stage; keeps final image minimal

**Do not use:** React as the primary framework, Framer Motion (React-coupled), styled-components (React-specific JS runtime), Gatsby (maintenance mode), Next.js (SSR/hybrid overhead for no gain on a 5-page static site), Three.js in Phase 1 (defer to Phase 2+).

### Expected Features

Every show page must answer "I found this show — what is it, and where do I listen?" in one scroll. The homepage must signal "cinematic studio" in the first viewport.

**Must have (table stakes):**
- Per-show page with show artwork, atmospheric hero section, show description/premise
- Episode listing per show with titles, numbers, and listen links (Spotify, Apple Podcasts)
- Show-specific accent color identity (CSS custom property per route)
- Lore section per show (world-building prose or styled artifacts)
- About / Studio page — who Dark Studios is, the mission
- Global navigation (max 5 items), footer with social links
- Mobile-responsive across all pages
- Per-page OG images — without these, social sharing shows a blank card
- Readable body copy — WCAG AA contrast (4.5:1 minimum) at all times

**Should have (differentiators):**
- Atmospheric hero with parallax or subtle animation per show page — highest return single feature for brand impression
- Homepage show tease cards with cinematic hover states, not generic podcast cards
- Seamless dark scroll transitions between content sections
- Show-specific lore section styling (Capital Archive: redacted archive entries; Fabulas De Machina: codex/amber aesthetic; Ashen Archives: ash-stained fragment aesthetic)
- `prefers-reduced-motion` on all animation — required for accessibility

**Defer to v2+:**
- Latest episode callout (manual update burden; add after show cadence is established)
- Show in-universe artifacts / cryptic teasers (high atmospheric value; requires copywriting; Phase 2 content addition)
- Full GSAP animation choreography (add after layout and content are stable)
- Three.js WebGL fog/particle scenes (strong differentiator; real implementation cost; Phase 2+)

**Anti-features (never build):**
- Embedded audio players (redirect to platform apps instead)
- Light mode toggle (Dark Studios is darkness; dilutes brand)
- Social media feed embeds (destroys performance)
- Auto-playing audio or video
- Infinite scroll episode feed

### Architecture Approach

The architecture is a multi-page static site with a shared design system and per-show content driven by Astro Content Collections. Each show page is generated at build time from `src/pages/shows/[slug].astro` via `getStaticPaths()`, populated from a typed Markdown schema in `src/content/shows/`. The shared `ShowLayout` component accepts a typed show data object and renders the complete show experience, eliminating per-show duplication. All three show pages use identical layout, differentiated only by content and their `accentColor` CSS custom property injected at the page root.

Build order is strict: theme tokens first (everything depends on them), then `BaseLayout`, then `ShowLayout`, then individual pages. Never build page-level work before the layout it depends on is stable.

**Major components:**
1. `BaseLayout` (`src/layouts/BaseLayout.astro`) — HTML shell, `<head>`, meta tags, global CSS, font loading; wraps every page
2. `ShowLayout` (`src/layouts/ShowLayout.astro`) — most critical shared component; accepts typed show prop; renders hero, lore, episodes, platform links for all three shows
3. `AtmosphericHero` (`src/components/AtmosphericHero.astro`) — full-viewport section with background image, overlay, title reveal; used by ShowLayout and HomeLayout
4. `EpisodeRow` (`src/components/EpisodeRow.astro`) — single episode with title, number, duration, external listen link
5. `ShowCard` (`src/components/ShowCard.astro`) — cinematic card for homepage show grid
6. `FogOverlay` (`src/components/FogOverlay.astro`) — CSS/canvas atmospheric grain; intensity controlled via prop
7. Content schema (`src/content/config.ts`) — Zod-validated frontmatter schema; build fails on missing fields; defines the Show type that ShowLayout and ShowCard depend on

**Critical patterns:**
- CSS custom properties for all theme values (`--color-void`, `--color-surface`, `--shadow-deep`) — never hardcoded hex in components
- Per-show accent color injected as `style="--show-accent: {accentColor}"` at page root
- Namespaced image directories: `public/images/shows/the-capital-archive/cover.webp`

### Critical Pitfalls

1. **React as the primary framework** — ships 200–400 KB JS to render five static pages; destroys LCP; the atmospheric hero becomes a blank hydration delay. Use Astro. This decision is expensive to reverse once committed.

2. **Carrying over `overflow: hidden` from the current terminal site** — the existing `index.html` sets this on `body` to lock the terminal frame. If it migrates to the new codebase, the multi-page site will not scroll and all scroll-triggered animations will fire on load. Write new global CSS from scratch; audit every rule from the old site before carrying it forward.

3. **Atmospheric images without a performance budget** — hero images as 2–4 MB JPEG/PNG make mobile LCP 8+ seconds. Establish the WebP/AVIF pipeline in Phase 1 before any image asset is added. Hero images target ≤150 KB at 1440px, ≤60 KB at 390px. Use Astro's `<Image>` component for automatic srcset generation.

4. **Low contrast text on dark backgrounds** — atmospheric dark palettes routinely produce contrast ratios below 3:1 for body text, failing WCAG AA and making show descriptions illegible. Define contrast-verified text tokens in `theme.css` before building any component. Use off-white (`#E8E0D0`, `#C8BFB0`) on near-black, not gray. Add dark scrim/gradient overlay on all text-over-image situations.

5. **nginx SPA fallback breaking multi-page output** — the current `try_files $uri $uri/ /index.html` was written for a single-page site. Update to `try_files $uri $uri.html $uri/index.html =404` before the first multi-page build ships. Otherwise deep links return homepage HTML at show page URLs and SEO crawlers index homepage content at every route.

**Moderate pitfalls to track:**
- GSAP added before content structure is stable (defer animation polish to Phase 4)
- Missing OG images on show pages (OG image is a required deliverable per show page)
- No `prefers-reduced-motion` (bake in from the first animation written)
- Font loading causing layout shift (self-host display fonts; `font-display: optional` for decorative faces)

---

## Implications for Roadmap

The architecture has a strict dependency order that directly maps to phases. Theme tokens must exist before components. Components must be stable before pages. Pages must exist before animation polish. Infrastructure must be validated before any page ships to production.

### Phase 1: Foundation — Framework, Design System, and Infrastructure

**Rationale:** Every other phase depends on this. Theme tokens are used by all components. The Dockerfile must be validated before anything ships. The `overflow: hidden` bug and contrast failure modes are silent and will corrupt all downstream phases if not caught here. This phase produces no visible pages — it produces the substrate everything else is built on.

**Delivers:**
- Astro project scaffolded with Tailwind CSS v4 and GSAP installed
- `theme.css` with contrast-verified color tokens, shadow values, typography scale
- `global.css` written from scratch (zero carry-over from index.html)
- `BaseLayout.astro` with font loading (Fontsource), meta tags, OG image slot
- `Header.astro` and `Footer.astro` as Astro components (zero JS shipped)
- Updated Dockerfile (two-stage: node:20-alpine → nginx:alpine, copies `dist/`)
- nginx.conf updated to `try_files $uri $uri.html $uri/index.html =404`
- WebP image pipeline established and size budgets documented

**Pitfalls avoided:** Pitfall 1 (wrong framework), Pitfall 12 (overflow hidden), Pitfall 4 (contrast tokens before components), Pitfall 5 (nginx routing correct before go-live), Pitfall 3 (image pipeline before images are added)

**Research flag:** Standard patterns — no additional research needed.

---

### Phase 2: Content Architecture — Show Schema and ShowLayout

**Rationale:** `ShowLayout` is the highest-reuse, highest-risk component. It serves all three show pages. Building it against real content before any show page ships validates the schema design. Content Collections must exist before any show page can be generated. This phase prevents the anti-pattern of three separately-coded show pages that immediately diverge.

**Delivers:**
- `src/content/config.ts` with Zod schema (title, slug, tagline, coverImage, accentColor, platforms, episodes array)
- Three show Markdown files with placeholder content
- `ShowLayout.astro` with per-show accent color injection via CSS custom property
- `AtmosphericHero.astro` (static — no GSAP yet; parallax deferred to Phase 4)
- `EpisodeRow.astro` with title, number, duration, external listen link
- `[slug].astro` dynamic route with `getStaticPaths()` — generates all three show pages at build

**Features addressed:** Per-show page, episode listing, listen links, show description, show-specific color identity

**Pitfalls avoided:** Pitfall 10 (content collections from day one), Anti-Pattern 1 (no per-show page files), Anti-Pattern 2 (ShowLayout abstraction established)

**Research flag:** Standard patterns — Astro Content Collections with Zod and `getStaticPaths()` are stable, production-grade APIs. No additional research needed.

---

### Phase 3: Page Content — Show Pages, Homepage, About

**Rationale:** With the design system and ShowLayout stable, all three show pages can be populated with real content. Homepage and About follow the same component primitives. OG images are a required deliverable for each page — not deferred — because they are what makes each page complete for social sharing.

**Delivers:**
- All three show pages with real content: atmospheric hero, lore section, episode list, platform links, show-specific palette
- Show lore sections styled per show identity (Capital Archive: archive/redacted aesthetic; Fabulas De Machina: codex/amber aesthetic; Ashen Archives: ash fragment aesthetic)
- Homepage with full-bleed atmospheric hero, show tease card grid, studio identity statement
- About page with studio mission, vision, optional team section
- `FogOverlay.astro` for static atmospheric grain (CSS `@keyframes`, no GSAP)
- Per-page OG images (1200×630, show key art + title + tagline)
- All hero images as WebP/AVIF within performance budget

**Features addressed:** All table-stakes features, lore sections, show-specific color identity, OG images, studio identity statement, homepage show tease, atmospheric heroes (static)

**Pitfalls avoided:** Pitfall 3 (all images WebP within budget), Pitfall 8 (OG images shipped with pages)

**Research flag:** No technical research needed. Show identity copy (lore text, taglines, episode data, listen URLs) must be written before this phase can complete — coordinate with show owners early. Show key art must exist as WebP within size budgets before Phase 3.

---

### Phase 4: Animation Polish — GSAP Scroll Reveals and Atmospheric Motion

**Rationale:** Animation is deferred to last deliberately. Content and layout must be stable before scroll trigger offsets, timeline choreography, and entrance sequences are written. This phase has no content risk — it only enhances a complete, functional site.

**Delivers:**
- GSAP ScrollTrigger parallax on show page hero images
- Staggered entrance reveals for show cards on homepage
- Fog/grain animation upgrades where CSS `@keyframes` are insufficient
- Seamless dark fade transitions between content sections
- `prefers-reduced-motion` media query on every animation declaration
- GSAP `window.matchMedia('prefers-reduced-motion')` check before any JS animation sequence

**Features addressed:** Atmospheric hero parallax, subtle ambient animation, seamless dark scrolling transitions

**Pitfalls avoided:** Pitfall 2 (transform and opacity only — no layout property animation), Pitfall 7 (GSAP deferred until layout stable), Pitfall 9 (prefers-reduced-motion baked in)

**Research flag:** Flag for phase-level research if the team is new to GSAP. GSAP ScrollTrigger initialization inside Astro `<script>` blocks has nuances (module script vs. inline, DOM ready timing, `astro:page-load` event if View Transitions are ever added). Well-documented but worth a targeted spike.

---

### Phase 5: Deployment Validation and QA

**Rationale:** The nginx change must be smoke-tested against all routes in production before the rebrand is live. Kubernetes rolling update behavior with a multi-page static site differs from the current single-file setup — explicit validation prevents the silent failure where cached routes return wrong HTML after a rollout.

**Delivers:**
- Smoke test for all routes (`/`, `/about`, `/shows/the-capital-archive`, `/shows/fabulas-de-machina`, `/shows/the-ashen-archives`)
- Lighthouse CI run for all pages (target ≥90 Performance, Accessibility, SEO)
- Contrast audit against WCAG AA for all text/background combinations
- Mobile device testing
- GitHub Actions workflow updated to trigger on `src/**` and `public/**` path changes

**Pitfalls avoided:** Pitfall 5 (nginx routing validated end-to-end), Pitfall 4 (contrast audit), Pitfall 3 (performance budget verified via Lighthouse CI)

**Research flag:** Standard QA patterns. No research needed.

---

### Phase Ordering Rationale

- Phase 1 before everything: contrast tokens, global CSS, and Docker/nginx infrastructure are depended on by all subsequent work. Catching `overflow: hidden` and nginx routing in Phase 1 prevents them from silently corrupting Phases 2–3.
- Phase 2 before Phase 3: ShowLayout and Content Collections schema must exist before show pages can be populated. Building ShowLayout against one show (The Capital Archive) validates the prop contract before applying to the other two.
- Phase 3 before Phase 4: GSAP animation requires stable layout. Scroll trigger offsets against a layout still changing is throw-away work.
- Phase 4 before Phase 5: QA must test the final animated state.
- Phase 5 is a gate, not an afterthought — specifically for nginx/K8s multi-page routing validation.

---

### Research Flags

**Needs research during planning:**
- **Phase 4 (Animation):** If the team has limited GSAP experience, a targeted spike on GSAP ScrollTrigger initialization inside Astro `<script>` blocks is warranted (module script vs. inline, DOM ready timing, `astro:page-load` event).

**Standard patterns (skip research-phase):**
- **Phase 1 (Foundation):** Astro scaffolding, Tailwind v4 integration, two-stage Dockerfile, nginx `try_files` — all well-documented.
- **Phase 2 (Content Architecture):** Astro Content Collections + Zod + `getStaticPaths()` — stable, production-grade, well-documented.
- **Phase 3 (Page Content):** Execution against established components — no research needed.
- **Phase 5 (QA):** Lighthouse CI, smoke testing — standard DevOps patterns.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Astro, GSAP, Tailwind v4 compose cleanly and are individually well-established. Version numbers (Astro ~5.6, GSAP ~3.12) are MEDIUM — verify via npm registry before scaffolding. |
| Features | HIGH | Table stakes and anti-features are well-established patterns for audio drama / podcast studio sites. Reference site live verification was unavailable (no WebFetch), but patterns are stable across the domain. |
| Architecture | HIGH | Astro Content Collections, `getStaticPaths()`, and static output are mature, production-grade APIs. Component hierarchy and build order are sound. No novel patterns — proven approach for this exact use case. |
| Pitfalls | HIGH | Critical pitfalls are grounded in direct analysis of the existing codebase and well-documented failure modes. The `overflow: hidden` finding and nginx routing issue are direct observations from the current codebase. |

**Overall confidence:** HIGH

### Gaps to Address

- **Typeface selection** (LOW confidence): STACK.md recommends Cinzel Decorative + Inter directionally, but this is aesthetic judgment, not a technical finding. Validate against brand direction before committing to font npm installs. Do not build typography around specific typefaces until confirmed.

- **Show content / lore copy** (out of research scope): All three show pages require real lore text, episode data (titles, numbers, durations, listen URLs), and show taglines. This content is a prerequisite for Phase 3. Identify and unblock content owners before Phase 3 begins.

- **Show key art / hero images** (out of research scope): The image pipeline is defined (WebP, size budgets, srcset), but the actual images must exist and be converted to spec. Establish whether existing show artwork needs to be created, sourced, or converted, and who owns that work.

- **Exact version numbers** (MEDIUM confidence): Verify `npm view astro version` and `npm view gsap version` before scaffolding. Training data versions are directionally correct but may be one minor version behind.

---

## Sources

### Primary (HIGH confidence)
- Astro documentation (astro.build) — static output, Content Collections, `getStaticPaths()`, Tailwind integration
- GSAP documentation (gsap.com) — ScrollTrigger, free tier, framework-agnostic usage
- WCAG 2.1 guidelines (w3.org/TR/WCAG21) — contrast requirements, reduced-motion success criteria
- Web Vitals documentation (web.dev/vitals) — LCP, CLS, TTI thresholds and targets
- nginx documentation — `try_files` directive behavior and static file serving
- Direct codebase analysis: `index.html`, `jobs.html`, `Dockerfile`, `nginx.conf`, `k8s/` manifests

### Secondary (MEDIUM confidence)
- Tailwind CSS v4 announcement (tailwindcss.com/blog/tailwindcss-v4) — CSS-first configuration, January 2025 release
- Fontsource (fontsource.org) — npm-installable Google Fonts, self-hosting pattern
- Domain knowledge: Audio drama network design patterns (Night Vale Presents, Rusty Quill/Magnus Archives, Realm, Headgum, Maximum Fun)
- Domain knowledge: Horror fiction brand design (NoSleep, Shudder, The Black Tapes, Limetown)

### Tertiary (LOW confidence, validate before committing)
- Typeface direction (Cinzel Decorative, Inter) — aesthetic judgment; verify against brand guidelines
- Specific npm version numbers for Astro (~5.6) and GSAP (~3.12) — training data; verify via npm registry

---
*Research completed: 2026-04-13*
*Ready for roadmap: yes*
