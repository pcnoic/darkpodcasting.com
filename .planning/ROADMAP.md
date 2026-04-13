# Roadmap: Dark Studios — Website Rebrand

## Overview

The rebrand replaces a terminal-aesthetic single-page site with a cinematic multi-page studio presence for Dark Studios. Work proceeds in strict dependency order: infrastructure and design system first (everything else depends on contrast tokens and the nginx routing fix), then the shared show architecture (ShowLayout drives all three show pages), then all page content (homepage, about, and fully-populated show pages), then animation polish (GSAP deferred until layout is stable), and finally deployment QA (smoke-test every route, run Lighthouse CI, verify the nginx change end-to-end before going live).

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation** - Astro scaffold, design system, BaseLayout, global nav/footer, updated Dockerfile and nginx config
- [ ] **Phase 2: Show Architecture** - Content Collections schema, ShowLayout, dynamic show routes for all three productions
- [ ] **Phase 3: Page Content** - All three show pages fully populated, homepage, About page, OG images
- [ ] **Phase 4: Animation Polish** - GSAP entrance reveals, ScrollTrigger scroll effects, prefers-reduced-motion coverage
- [ ] **Phase 5: QA and Deployment** - Smoke test all routes, Lighthouse CI, contrast audit, mobile verification

## Phase Details

### Phase 1: Foundation
**Goal**: The project has a working Astro scaffold with a contrast-verified dark design system, shared layout components, and correct Docker/nginx infrastructure — every subsequent phase builds on this substrate
**Depends on**: Nothing (first phase)
**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04, BRAND-03
**Success Criteria** (what must be TRUE):
  1. Running `astro build` produces a static `dist/` directory that nginx can serve without errors
  2. All color and typography tokens in `theme.css` pass WCAG AA contrast verification (4.5:1 minimum for body text)
  3. The Dockerfile builds successfully in two stages (node builder → nginx alpine) and the resulting image serves the site
  4. Global Header and Footer components render consistently on every page, with no "Dark Podcasting" references anywhere in shared layout
  5. nginx `try_files` directive resolves `/shows/the-capital-archive` and other deep paths to the correct HTML file, not a 404 or homepage fallback
**Plans**: TBD

### Phase 2: Show Architecture
**Goal**: A single shared ShowLayout component and typed Content Collections schema can generate all three show pages from Markdown files — no per-show duplication
**Depends on**: Phase 1
**Requirements**: SHOW-01, SHOW-02, SHOW-03, SHOW-04, SHOW-05
**Success Criteria** (what must be TRUE):
  1. Running `astro build` generates three distinct show pages at `/shows/the-capital-archive`, `/shows/fabulas-de-machina`, and `/shows/the-ashen-archives`
  2. Each show page renders a full-bleed hero section with the show's accent color applied via CSS custom property
  3. Each show page renders a lore section with the show's premise text
  4. Each show page renders an ordered episode list with external listen links
  5. Each show page has a unique OG image tag in `<head>` that resolves to a valid 1200×630 image asset
**Plans**: TBD
**UI hint**: yes

### Phase 3: Page Content
**Goal**: All five pages — three show pages, homepage, and About — are fully populated with real content and the "Dark Studios" identity is complete across the entire site
**Depends on**: Phase 2
**Requirements**: HOME-01, HOME-02, HOME-03, ABOUT-01, ABOUT-02, ABOUT-03, ABOUT-04, BRAND-01
**Success Criteria** (what must be TRUE):
  1. The homepage opens with a full-bleed Dark Studios hero — the studio name, tagline, and atmospheric imagery are visible in the first viewport with no reference to "Dark Podcasting"
  2. The homepage show grid presents all three shows with links that navigate to each show's dedicated page
  3. A visitor on the homepage can reach the About page via a studio mission teaser or navigation
  4. The About page presents the Dark Studios mission statement, an overview of all three shows with links, team credits, and contact or social links
  5. Every page across the site uses "Dark Studios" identity — no remaining "Dark Podcasting" references in page titles, headings, meta tags, or body copy
**Plans**: TBD
**UI hint**: yes

### Phase 4: Animation Polish
**Goal**: GSAP entrance animations and ScrollTrigger scroll effects reinforce the cinematic atmosphere across all pages without degrading performance or blocking layout
**Depends on**: Phase 3
**Requirements**: ANIM-01, ANIM-02, ANIM-03
**Success Criteria** (what must be TRUE):
  1. Page elements (show cards, hero text, lore sections) animate in on load with staggered fade/reveal transitions that feel cinematic rather than utilitarian
  2. Show page hero sections exhibit a GSAP ScrollTrigger parallax or fog reveal effect as the user scrolls past the hero
  3. All animations are absent when `prefers-reduced-motion: reduce` is active — the page is fully usable with zero motion
  4. Animations do not cause measurable layout shift (CLS remains at 0) and hero images still achieve target LCP
**Plans**: TBD

### Phase 5: QA and Deployment
**Goal**: Every route resolves correctly in production, all pages pass Lighthouse thresholds, contrast and mobile layout are verified — the site is ready to go live
**Depends on**: Phase 4
**Requirements**: FOUND-04 (end-to-end verification), BRAND-02
**Success Criteria** (what must be TRUE):
  1. All five routes (`/`, `/about`, `/shows/the-capital-archive`, `/shows/fabulas-de-machina`, `/shows/the-ashen-archives`) return 200 HTTP responses with correct page content when tested against the production nginx/K8s environment
  2. Lighthouse CI scores ≥90 for Performance, Accessibility, and SEO on every page
  3. All text/background combinations across the site pass WCAG AA contrast (4.5:1 for body text, 3:1 for large text)
  4. All pages are usable and visually correct on a 390px-wide mobile viewport with no horizontal overflow or broken layouts
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 0/TBD | Not started | - |
| 2. Show Architecture | 0/TBD | Not started | - |
| 3. Page Content | 0/TBD | Not started | - |
| 4. Animation Polish | 0/TBD | Not started | - |
| 5. QA and Deployment | 0/TBD | Not started | - |
