# Requirements: Dark Studios — Website Rebrand

**Defined:** 2026-04-13
**Core Value:** The first impression must be unmistakably atmospheric, dark, and intriguing — a visitor should feel Dark Studios' identity the moment the page loads.

## v1 Requirements

### Foundation

- [ ] **FOUND-01**: Site is built with Astro 5 and exports static HTML/CSS/JS files compatible with nginx static hosting
- [ ] **FOUND-02**: Dark design system with CSS custom properties for palette, typography, and spacing is defined and contrast-verified against WCAG AA before any component ships
- [ ] **FOUND-03**: Dockerfile uses a multi-stage build (Node builder → nginx alpine) that produces equivalent behavior to current deployment
- [ ] **FOUND-04**: Nginx configuration correctly routes multi-page paths so all show pages and deep links resolve without 404

### Branding

- [ ] **BRAND-01**: All pages use "Dark Studios" identity — no remaining references to "Dark Podcasting"
- [ ] **BRAND-02**: Site is fully mobile-responsive across all pages and screen sizes
- [ ] **BRAND-03**: Global navigation and footer are consistent across all pages

### Shows

- [ ] **SHOW-01**: Each of the three shows (The Capital Archive, Fabulas De Machina, The Ashen Archives) has its own dedicated page at a unique URL
- [ ] **SHOW-02**: Each show page has an atmospheric full-bleed hero section with show-specific accent color and mood imagery
- [ ] **SHOW-03**: Each show page has a lore / about section presenting the show's premise, world, and tone in prose
- [ ] **SHOW-04**: Each show page has an ordered episode list with episode titles and listen links to podcast platforms
- [ ] **SHOW-05**: Each show page has a unique OG image for social sharing previews

### Homepage

- [ ] **HOME-01**: Homepage opens with a full-bleed Dark Studios identity hero section with studio tagline
- [ ] **HOME-02**: Homepage presents all three shows in a discovery grid or layout linking to individual show pages
- [ ] **HOME-03**: Homepage includes a brief studio mission teaser linking to the full About page

### About

- [ ] **ABOUT-01**: About page presents the Dark Studios mission statement
- [ ] **ABOUT-02**: About page includes an overview of all three shows with links to their pages
- [ ] **ABOUT-03**: About page includes team credits or studio attribution
- [ ] **ABOUT-04**: About page includes contact information and social media links

### Animation

- [ ] **ANIM-01**: Page elements have entrance animations (CSS/GSAP fade-in, stagger) on initial load that reinforce the atmospheric mood
- [ ] **ANIM-02**: Show page heroes have GSAP ScrollTrigger-based scroll effects (parallax, fog, reveal) that deepen the cinematic atmosphere
- [ ] **ANIM-03**: Scroll-based animations are non-blocking — do not degrade LCP or cause layout shift

## v2 Requirements

### Visual Effects

- **VFX-01**: Three.js / WebGL atmospheric hero effects (particles, depth, fog simulation) on homepage
- **VFX-02**: Per-show ambient background animations (not scroll-triggered, ambient loop)

### Content

- **CONT-01**: Teaser / placeholder pages for upcoming projects (Dark Studios pipeline)
- **CONT-02**: Newsletter or mailing list signup for new releases

### Jobs

- **JOBS-01**: Revamped jobs / casting page aligned with Dark Studios brand

## Out of Scope

| Feature | Reason |
|---------|--------|
| Embedded audio players | Anti-feature — direct to podcast platform; players harm atmosphere and add performance cost |
| CMS or backend | Static site deployment maintained — content managed via markdown files |
| Server-side rendering | nginx static hosting requires static export only |
| E-commerce / subscriptions | No monetization in scope |
| New show teasers (v1) | Focus on existing three shows; new shows are v2 |
| Jobs / casting page (v1) | Deferred to v2 |
| React as framework | Wrong tool — 5 static pages don't warrant RSC/App Router overhead |
| Autoplay audio/video | Breaks atmospheric first impression, violates browser autoplay policies |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01 | Phase 1 | Pending |
| FOUND-02 | Phase 1 | Pending |
| FOUND-03 | Phase 1 | Pending |
| FOUND-04 | Phase 1 | Pending |
| BRAND-03 | Phase 1 | Pending |
| SHOW-01 | Phase 2 | Pending |
| SHOW-02 | Phase 2 | Pending |
| SHOW-03 | Phase 2 | Pending |
| SHOW-04 | Phase 2 | Pending |
| SHOW-05 | Phase 2 | Pending |
| HOME-01 | Phase 3 | Pending |
| HOME-02 | Phase 3 | Pending |
| HOME-03 | Phase 3 | Pending |
| ABOUT-01 | Phase 3 | Pending |
| ABOUT-02 | Phase 3 | Pending |
| ABOUT-03 | Phase 3 | Pending |
| ABOUT-04 | Phase 3 | Pending |
| BRAND-01 | Phase 3 | Pending |
| ANIM-01 | Phase 4 | Pending |
| ANIM-02 | Phase 4 | Pending |
| ANIM-03 | Phase 4 | Pending |
| BRAND-02 | Phase 5 | Pending |

**Coverage:**
- v1 requirements: 22 total
- Mapped to phases: 22
- Unmapped: 0

---
*Requirements defined: 2026-04-13*
*Last updated: 2026-04-13 — traceability populated after roadmap creation*
