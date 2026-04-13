# Domain Pitfalls

**Domain:** Atmospheric creative studio / podcast website — visually-heavy, animation-rich static site
**Project:** Dark Studios website rebrand (darkpodcasting.com)
**Researched:** 2026-04-13
**Confidence:** HIGH (well-established patterns in static site, animation performance, and creative studio web dev)

---

## Critical Pitfalls

Mistakes that cause rewrites, major regressions, or destroy the atmospheric effect entirely.

---

### Pitfall 1: Choosing a Framework for "Better Animations" Without Understanding the Trade-Off

**What goes wrong:** The team picks React because it "supports animations better," ships a full SPA with client-side routing, and the site ends up with a 200–400 KB JS bundle just to render five mostly-static pages. Time to first meaningful paint is 3–5 seconds. The atmospheric hero — the most important moment of the first impression — is behind a blank white flash or hydration delay.

**Why it happens:** Animation libraries (Framer Motion, GSAP React integration) are often cited in React tutorials. The association between "rich animations" and "React" is common but confused. Animations live in CSS or imperative JS — the framework is irrelevant to animation capability.

**Consequences:**
- Performance score collapses (LCP 3–5 s on mobile)
- The "feel" the design aims for is destroyed at the entry point
- Refactor to Astro mid-project costs 2–3 days

**Prevention:** Use Astro. For five static pages with atmospheric content and no user login, shopping cart, or real-time state, Astro ships zero JS by default and adds only what the page requires. GSAP, CSS animations, and even React islands work inside Astro without the full SPA cost. The decision is still TBD — this pitfall is the primary argument for Astro over React.

**Detection — warning signs:**
- Bundle analyzer shows >80 KB JS for a page with no forms or data fetching
- Lighthouse TTI (Time to Interactive) exceeds 3 s on simulated 4G
- A blank frame is visible before the hero renders

**Phase to address:** Framework selection phase (Phase 1 / foundation). Once React is committed to, this becomes expensive to undo.

---

### Pitfall 2: Animating Layout Properties (width, height, top, left)

**What goes wrong:** Hero entrance animations, fog drift effects, and show-card reveals are implemented by animating `width`, `height`, `margin`, `top`, or `left`. The browser triggers full layout recalculation and paint on every frame. On mid-range Android devices this drops to 15–20 fps. The cinematic mood becomes a stuttering slideshow.

**Why it happens:** These properties are intuitive. "I want the element to grow" → animate `width`. The GPU-composited properties (`transform`, `opacity`) are less obvious.

**Consequences:**
- Jank on every mobile device below flagship tier
- Battery drain on mobile
- Users leave during the "atmospheric entrance" that was meant to hook them

**Prevention:** All motion must use only `transform` (translate, scale, rotate) and `opacity`. For fog/parallax effects, use `transform: translateZ(0)` or `will-change: transform` to promote elements to their own compositor layer. Never animate `filter` on large elements at 60 fps — pre-render blurred/fogged variants as image assets instead.

**Detection — warning signs:**
- Chrome DevTools Performance panel shows "Layout" events during animation
- Dropped frames visible in the FPS meter during scroll or entrance
- "Recalculate Style" or "Layout" takes >1 ms per frame

**Phase to address:** Animation implementation phase. Establish a one-rule constraint: "transform and opacity only" before any animation is written.

---

### Pitfall 3: Atmospheric Imagery Without a Performance Budget

**What goes wrong:** High-quality atmospheric imagery — fog, dark environments, cinematic stills — is used in large JPEG/PNG format. Each show page hero is a 2–4 MB image. The homepage loads three of them simultaneously. Mobile LCP is 8+ seconds. The "atmospheric first impression" is an empty black screen followed by images loading line by line.

**Why it happens:** The images look great in Figma or on a development machine with fast networking. The designer and developer see instant loads. The production visitor on mobile 4G does not.

**Consequences:**
- Google Core Web Vitals failure (LCP red)
- High bounce rate precisely when users are deciding if they care
- The entire atmospheric investment in imagery is negated

**Prevention:**
- All hero/atmospheric images must be WebP or AVIF, not JPEG/PNG
- Images must have explicit `width` and `height` to prevent CLS (Cumulative Layout Shift)
- Hero images must be `loading="eager"` with `fetchpriority="high"` — not lazy-loaded
- Below-fold show imagery must be lazy-loaded
- Target: hero image ≤150 KB at 1440px, ≤60 KB at 390px (mobile)
- Use `<picture>` with `srcset` for responsive delivery
- Astro's `<Image>` component handles this automatically — one more point for Astro

**Detection — warning signs:**
- Any atmospheric image file >300 KB before optimization
- No `.webp` or `.avif` files in the images directory (currently only `favicon.ico` exists — new assets need the pipeline from day one)
- LCP element in Lighthouse is an image

**Phase to address:** Asset pipeline setup (Phase 1/2). Retroactively converting images after show pages are built is tedious and causes layout rework.

---

### Pitfall 4: Dark Design + Poor Contrast = Accessibility Failure + Unreadable Content

**What goes wrong:** The atmospheric dark palette (near-black backgrounds, desaturated overlays, muted text) produces contrast ratios below 3:1 for body text and below 4.5:1 for UI text. Show descriptions become illegible. Episode titles disappear against the dark show artwork. The site looks intentionally moody but fails WCAG AA, gets flagged by screen readers, and is literally unreadable in bright ambient light.

**Why it happens:** Designers preview dark UIs in dark-mode macOS environments on calibrated displays. The contrast looks fine. A visitor checking on their phone in daylight or a user with low vision sees nothing.

**Consequences:**
- Lighthouse accessibility score drops below 70
- SEO ranking penalty (Google uses accessibility signals)
- Users cannot read episode descriptions — direct content failure
- Legal exposure in some jurisdictions

**Prevention:**
- Minimum contrast 4.5:1 for all body text (WCAG AA)
- Minimum contrast 3:1 for large headings (≥24px) and UI elements
- "Dark atmospheric" does not require low contrast — use off-white (`#E8E0D0`, `#C8BFB0`) on very dark backgrounds, not gray on near-black
- Test every text-on-background combination with a contrast checker during design
- Text over imagery must have a dark scrim/gradient overlay — never raw text on a photographic background

**Detection — warning signs:**
- Any text color lighter than `#AAAAAA` on a background darker than `#111111` without a contrast check
- Lighthouse accessibility score below 80
- `axe` or `WAVE` tool reporting contrast errors

**Phase to address:** Design system phase (before any page implementation). Contrast values must be in the design tokens, not per-component decisions.

---

### Pitfall 5: SPA-Style nginx Routing Breaking Static Multi-Page Structure

**What goes wrong:** The nginx config already has `try_files $uri $uri/ /index.html` — a single-page app fallback. If the framework outputs multiple HTML files (`/shows/capital-archive/index.html`) but nginx serves the root `index.html` for all unknown paths, deep-linking works by accident in some cases and not others. After a Kubernetes rollout swaps the pod, cached routes return 200 with the wrong HTML. Show pages appear to work during development (file:// or dev server) but 404 or mis-serve in production.

**Why it happens:** The nginx config was written for the current single-page site. It was not designed for a multi-page static output. Nobody updates it during the framework migration because "it was already working."

**Consequences:**
- Show pages return the homepage HTML on direct navigation or refresh
- SEO crawlers index the homepage content at show page URLs
- Kubernetes readiness probes may pass while actual pages are broken

**Prevention:**
- When moving to Astro/React static export, update nginx to serve actual files first: `try_files $uri $uri.html $uri/index.html =404`
- Remove the SPA fallback entirely or restrict it only if a true SPA mode is chosen
- Add smoke-test requests to each show page URL in the Kubernetes deployment health check
- The Dockerfile must be updated to copy the build output directory, not individual HTML files

**Detection — warning signs:**
- `curl https://darkpodcasting.com/shows/capital-archive` returns homepage `<title>` content
- nginx access log shows 200 for paths that should return specific show pages
- The existing Dockerfile (`COPY index.html` + `COPY jobs.html`) pattern will break with multi-page output — it is a direct precursor to this failure

**Phase to address:** Deployment/infrastructure phase. Must be addressed before any show page is shipped to production.

---

## Moderate Pitfalls

---

### Pitfall 6: Font Loading Causing Layout Shift and Flash of Invisible Text

**What goes wrong:** Jersey 10 Charted and JetBrains Mono are loaded from Google Fonts. On slow connections, headings render in a fallback sans-serif, then snap to the display font when it loads. The atmospheric typography — often the most distinctive part of a dark cinematic identity — flashes to something generic at first render. CLS score increases. The loading experience undermines the brand.

**Prevention:**
- Use `font-display: swap` (already implied by Google Fonts `&display=swap`) — but also set a matching fallback font stack that approximates the metrics of the primary font to reduce the shift magnitude
- For hero headings using a display font like Jersey 10 Charted, consider self-hosting: download the subset WOFF2, serve from the same origin, and use `<link rel="preload">` for the font file
- `font-display: optional` is worth considering for decorative display fonts — the page renders without them, no shift
- Subset the fonts to only the characters actually used (Latin + numbers) using `glyphhanger` or Fonttools

**Detection — warning signs:**
- CLS score above 0.1 in Lighthouse
- Visible font swap during slow-network throttling in DevTools
- Google Fonts request visible as a render-blocking resource

**Phase to address:** Typography/design system phase (Phase 1/2).

---

### Pitfall 7: Scroll-Triggered Animation Libraries Added Before Core Content

**What goes wrong:** GSAP ScrollTrigger, AOS, or Framer Motion scroll animations are added in Phase 1 as part of "making it atmospheric." Every element fades in on scroll. The team spends a week on animation choreography. Then content structure changes in Phase 2 when show pages are built — all the scroll positions, trigger offsets, and animation sequences must be reconfigured. Animation implementation done before content structure is locked is throw-away work.

**Prevention:**
- Implement animations in the final phase, after content and layout are stable
- Use CSS `@keyframes` + `animation-play-state` or `Intersection Observer` for simple entrance effects in early phases — these are cheap to discard
- Reserve GSAP or Framer Motion for the polish phase when you know what you're polishing

**Detection — warning signs:**
- Animation code written before show page content structure is finalized
- ScrollTrigger offsets hardcoded to pixel values against a layout that is still changing
- Animation rework appears in sprint tasks in Phase 2 and 3

**Phase to address:** Defer animation polish to final phase. Placeholder transitions only during development.

---

### Pitfall 8: Open Graph Images Neglected on Show Pages

**What goes wrong:** The homepage has OG/Twitter meta tags (visible in the current `index.html`). New show pages are built without OG images or with generic fallbacks. When a listener shares a show page link on social, it renders a blank card or the homepage image. The atmospheric show artwork — which is the primary hook for new listeners — never appears in social shares.

**Prevention:**
- Each show page must have a dedicated OG image: 1200×630 px, show key art, show title, tagline
- Add OG meta tags as a required checklist item for each show page before it ships
- Static OG images are sufficient — no need for dynamic generation at this scale

**Detection — warning signs:**
- Show pages missing `og:image` meta tag
- `og:image` pointing to the same generic image used by the homepage
- LinkedIn/Twitter/Facebook debugger tools showing blank card preview

**Phase to address:** Show page implementation phase. Each show page is incomplete without its OG image.

---

### Pitfall 9: No `prefers-reduced-motion` Support

**What goes wrong:** Atmospheric animations — parallax, fog drift, hero fades — run at full intensity for users who have set "reduce motion" in their OS accessibility preferences. For users with vestibular disorders, motion sickness, or epilepsy, heavy parallax and persistent motion are not just unpleasant — they can cause physical symptoms. This is a WCAG 2.1 requirement (Success Criterion 2.3.3 at AAA, but 2.3.1 AA for seizure triggers).

**Prevention:**
- Every animation and parallax effect must be wrapped in a `@media (prefers-reduced-motion: reduce)` query that disables or significantly reduces motion
- In JavaScript animation code (GSAP), check `window.matchMedia('(prefers-reduced-motion: reduce)').matches` before initializing sequences
- The reduced-motion version should still feel atmospheric — use static opacity, no motion, same dark imagery

**Detection — warning signs:**
- No `prefers-reduced-motion` in any CSS file
- Animation JS does not check the media query
- Lighthouse accessibility audit flagging motion-related issues

**Phase to address:** Animation implementation phase. Bake in from the start, not as an afterthought.

---

## Minor Pitfalls

---

### Pitfall 10: Hardcoded Episode Data in Component Code

**What goes wrong:** Show episode lists (title, date, description, link) are hardcoded directly in JSX/HTML components. Adding a new episode requires editing a source file, running a build, and deploying. The site is static by design (no CMS in scope) — but the content update process should still not require a developer.

**Prevention:**
- Store episode data in JSON or Markdown files under a `content/` directory (Astro's content collections are ideal for this)
- Components read from data files; the build generates the HTML
- A non-developer can add an episode by adding a JSON entry and triggering a build
- This is not a CMS — it is just separating data from markup, which is low-effort and prevents future brittleness

**Detection — warning signs:**
- Episode data embedded directly in `.astro` or `.jsx` component files
- Show page requires a developer to add episodes
- Three show pages have three separate hard-coded episode arrays with no shared schema

**Phase to address:** Show page implementation phase (Phase 2/3). Establish the content/data pattern before building all three show pages.

---

### Pitfall 11: Kubernetes Deployment Complexity Creep

**What goes wrong:** The existing k8s config is minimal (base + overlays pattern). During the framework migration, the Dockerfile gets complicated — multi-stage builds, build args, environment variables for analytics IDs, etc. What was a simple `FROM nginx` copy becomes a fragile build pipeline that breaks on dependency version bumps or Node.js version mismatches.

**Prevention:**
- Keep the Dockerfile to two stages maximum: build stage (Node + framework CLI), serve stage (nginx)
- Pin the Node.js version in the build stage
- Do not pass environment variables at build time for a fully static site — there is no runtime, there are no secrets
- The nginx config is already correct for static files; resist adding middleware layers

**Detection — warning signs:**
- Dockerfile exceeds 30 lines
- Environment variables being baked into HTML at build time
- k8s overlays diverging significantly between environments for a static site with no backend

**Phase to address:** Infrastructure/deployment phase. Keep the container config as dumb as possible.

---

### Pitfall 12: `overflow: hidden` on `body` Breaking Atmospheric Scroll Effects

**What goes wrong:** The current `index.html` sets `overflow: hidden` on `body` (the terminal site uses this to prevent the content from scrolling past the fixed terminal frame). If this CSS carries over into the new framework codebase — either copy-pasted or via a CSS reset — scroll-triggered animations, parallax sections, and multi-section show pages will not scroll at all. This is a non-obvious silent failure.

**Prevention:**
- Audit every global CSS rule carried over from the current codebase
- Remove `overflow: hidden` from `body` in the new design unless there is a specific, documented reason to keep it for a particular page
- The current site's `overflow: hidden` is intentional for the terminal framing — it is not a neutral default

**Detection — warning signs:**
- Page does not scroll on mobile or desktop after framework setup
- Scroll-triggered animations fire on load rather than scroll
- `body` or `html` style inspection in DevTools shows `overflow: hidden`

**Phase to address:** Framework setup / global styles phase (Phase 1). First thing to check when the base layout is wired up.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|----------------|------------|
| Framework selection | Picking React for a 5-page static site (Pitfall 1) | Evaluate Astro first; default to React only if interactivity requirements exceed Astro islands |
| Global CSS / design system | Carrying over `overflow: hidden` and terminal-era resets (Pitfall 12) | Write new global CSS from scratch; do not copy from `index.html` |
| Design tokens / typography | Low contrast on atmospheric dark palette (Pitfall 4) | Verify every text/bg combination before any component is built |
| Asset pipeline | Adding atmospheric images without a performance budget (Pitfall 3) | Set image format and size targets before any image asset is added |
| Font loading | Display font causing layout shift (Pitfall 6) | Self-host Jersey 10 Charted; use `font-display: optional` |
| Show page implementation | Hardcoding episode data (Pitfall 10) | Use Astro content collections or JSON data files for all episode content |
| Show page implementation | Missing OG images (Pitfall 8) | OG image is a required deliverable for each show page |
| Animation implementation | Animating layout properties (Pitfall 2) | "transform and opacity only" rule, enforced in code review |
| Animation implementation | Missing `prefers-reduced-motion` (Pitfall 9) | Add the media query to every animation declaration |
| Animation implementation | Adding scroll animations before layout is stable (Pitfall 7) | Defer GSAP/Framer Motion to final polish phase |
| Deployment | nginx SPA fallback breaking multi-page output (Pitfall 5) | Update nginx config before first multi-page build ships to production |
| Deployment | Dockerfile complexity creep (Pitfall 11) | Two-stage max; no runtime env vars; pin Node version |

---

## Sources

- Analysis of existing codebase (`index.html`, `nginx.conf`, `Dockerfile`, `k8s/` structure) — HIGH confidence
- Web Vitals documentation (web.dev/vitals) — HIGH confidence (core metrics are stable)
- WCAG 2.1 guidelines (w3.org/TR/WCAG21) — HIGH confidence (normative standard)
- CSS compositing and rendering pipeline (MDN, Chrome DevTools documentation) — HIGH confidence
- Astro documentation patterns for static export and content collections — HIGH confidence
- nginx `try_files` directive behavior — HIGH confidence (documented, deterministic)
- Google Fonts `font-display` parameter behavior — HIGH confidence
