<!-- GSD:project-start source:PROJECT.md -->
## Project

**Dark Studios — Website Rebrand**

Dark Studios is an immersive audio drama and horror podcast studio. This project is a full visual overhaul and rebrand of darkpodcasting.com — retiring the terminal aesthetic in favor of a cinematic, atmospheric studio identity. The redesign introduces a JavaScript framework and individual show pages for each of the three existing productions.

**Core Value:** The first impression must be unmistakably atmospheric, dark, and intriguing — a visitor should feel Dark Studios' identity the moment the page loads.

### Constraints

- **Hosting:** Static site deployed via nginx in Kubernetes — framework output must be static-exportable
- **Domain:** darkpodcasting.com — URL unchanged, but branding transitions to Dark Studios
- **No backend:** No server-side rendering, no database — all content is built into the static output
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Framework
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Astro | 5.x (latest: ~5.6) | Static site framework, routing, component model | Purpose-built for content sites with full static output. Zero JS by default — only ships what you mark as interactive. File-based routing maps cleanly to the 5-page structure (home, 3 show pages, about). Native TypeScript, Vite-powered build. |
| Node.js | 20 LTS | Build runtime only (not deployed) | Astro's minimum supported runtime. Not present in final Docker image. |
### Styling
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Tailwind CSS | 4.x (v4.0 released Jan 2025) | Utility-first CSS | Tailwind v4 is CSS-first — configured via `@import "tailwindcss"` in a CSS file, no `tailwind.config.js` required for standard usage. Works natively with Astro via the `@astrojs/tailwind` integration (or direct PostCSS in v4). For dark atmospheric design, utility classes make iteration fast: `bg-black/90 backdrop-blur-lg shadow-[0_0_80px_rgba(0,0,0,0.9)]`. Scoped styles in Astro components complement Tailwind for component-specific overrides. |
### Animation
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| GSAP (GreenSock) | 3.x (latest: ~3.12) | Scroll-driven cinematic animations, entrance reveals, parallax | GSAP is framework-agnostic. It targets DOM elements directly — no React lifecycle dependency, no virtual DOM reconciliation. Works identically in Astro's `<script>` blocks. ScrollTrigger (bundled free) is the industry standard for scrubbed timeline animations tied to scroll position: parallax headers, fog reveal on show pages, title scrub effects. GSAP handles all timing edge cases (mobile scroll inertia, resize recalculation) that CSS animations and simple JS do not. |
| CSS animations + `@keyframes` | Native | Looping subtle effects (fog drift, particle float, ambient pulse) | For purely looping, non-interactive effects, CSS `@keyframes` with `animation` properties are more performant than JS-driven loops (run on compositor thread). Reserve for "always on" effects that never need to be scrubbed or programmatically controlled. |
### Fonts
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Fontsource | npm packages (per-font) | Self-hosted web fonts | Fontsource packages Google Fonts as npm modules. Self-hosting avoids the Google Fonts CDN DNS lookup, eliminates Google tracking concerns, and enables `font-display: swap` tuning. Fonts become build-time dependencies and are copied to the static output — no runtime CDN dependency. |
- Drop Jersey 10 Charted and JetBrains Mono as primary faces. They are terminal aesthetic — exactly what the rebrand is moving away from.
- Cinzel Decorative (serif, Roman, gothic weight) — for display headings and show titles. Evokes antiquity and dread. Available on Fontsource.
- Inter or DM Sans — for body copy and UI labels. Clean, readable on dark backgrounds at small sizes. Available on Fontsource.
- JetBrains Mono — retain only for episode timestamps, code-like metadata if used. Do not use as a primary face.
### Build & Deployment
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Multi-stage Dockerfile | — | Build inside Docker, serve with nginx | The current Dockerfile `COPY index.html` approach must change. Stage 1: `node:20-alpine` runs `astro build` to produce the `dist/` directory. Stage 2: `nginx:alpine` copies `dist/` to `/usr/share/nginx/html/`. Final image contains only nginx + static files — no Node.js runtime in production. Same image size profile as current. |
| Vite | 6.x (bundled with Astro) | Build tooling | Astro uses Vite internally. No separate Vite configuration needed for standard use. |
# Stage 1: Build
# Stage 2: Serve
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
## Installation
# Scaffold Astro project
# Astro integrations
# Animation
# Fonts (example — confirm typefaces in design phase)
# TypeScript (included with Astro by default)
# No additional install needed
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
## Sources
- Astro documentation (astro.build/docs) — static output, islands architecture, Tailwind integration
- GSAP documentation (gsap.com/docs) — ScrollTrigger, free tier capabilities
- Tailwind CSS v4 announcement (tailwindcss.com/blog/tailwindcss-v4) — CSS-first config, January 2025 release
- Fontsource (fontsource.org) — npm-installable Google Fonts
- Project files: `Dockerfile`, `nginx.conf`, `k8s/base/deployment.yaml`, `.github/workflows/deploy.yml` — confirms static nginx deployment, Docker multi-stage feasibility
- `index.html` — confirms current terminal aesthetic (green #00ff00, JetBrains Mono) being replaced
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, or `.github/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
