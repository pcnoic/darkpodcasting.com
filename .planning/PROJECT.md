# Dark Studios — Website Rebrand

## What This Is

Dark Studios is an immersive audio drama and horror podcast studio. This project is a full visual overhaul and rebrand of darkpodcasting.com — retiring the terminal aesthetic in favor of a cinematic, atmospheric studio identity. The redesign introduces a JavaScript framework and individual show pages for each of the three existing productions.

## Core Value

The first impression must be unmistakably atmospheric, dark, and intriguing — a visitor should feel Dark Studios' identity the moment the page loads.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Rebrand visual identity from "Dark Podcasting" to "Dark Studios" across all pages
- [ ] Atmospheric dark design — heavy blacks, shadow, cinematic mood; not terminal/hacker aesthetic
- [ ] Homepage that communicates the studio identity and surfaces the three shows
- [ ] Individual show page for The Capital Archive (lore, episodes, atmosphere)
- [ ] Individual show page for Fabulas De Machina (lore, episodes, atmosphere)
- [ ] Individual show page for The Ashen Archives (lore, episodes, atmosphere)
- [ ] About / Studio page — who Dark Studios is, the vision, the team
- [ ] Introduce a JavaScript framework (React or Astro) for better tooling, animations, and component reuse
- [ ] Mobile-responsive across all pages

### Out of Scope

- Jobs / casting page redesign — not prioritized for this milestone
- New show teasers / upcoming project pages — focus on existing three shows only
- CMS or backend — static site deployment maintained
- E-commerce or subscription features — no monetization in scope

## Context

- **Current site:** Vanilla HTML/CSS, single `index.html` + `jobs.html`. Terminal aesthetic — JetBrains Mono, green (#00ff00) on black. Minimal content.
- **Existing shows:** The Capital Archive, Fabulas De Machina, The Ashen Archives — immersive horror and audio drama podcasts
- **Deployment:** Docker + Kubernetes + nginx (existing infra at darkpodcasting.com — unchanged)
- **Fonts in use:** Jersey 10 Charted, JetBrains Mono (may be replaced or supplemented by rebrand)
- **Rebrand direction:** Dark atmosphere — heavy blacks, fog, shadow, ominous visual mood. More cinematic studio, less hacker terminal.

## Constraints

- **Hosting:** Static site deployed via nginx in Kubernetes — framework output must be static-exportable
- **Domain:** darkpodcasting.com — URL unchanged, but branding transitions to Dark Studios
- **No backend:** No server-side rendering, no database — all content is built into the static output

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Introduce a JS framework | Better animation capabilities, component reuse across show pages, improved DX | — Pending (React vs Astro TBD) |
| Atmospheric dark design over terminal aesthetic | Terminal look doesn't convey "real studio" — brand needs cinematic weight | — Pending |
| Individual show pages over cards | Each show deserves immersive space for lore, episodes, and atmosphere | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-13 after initialization*
