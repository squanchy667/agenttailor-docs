# T039 — Landing Page

| Field | Value |
|-------|-------|
| Phase | 8 — Polish & Launch |
| Agent Type | frontend-agent |
| Depends On | T001 |
| Blocks | T040 |
| Branch | `feat/T039-landing-page` |
| Status | PENDING |

## Objective
Build a marketing landing page with a hero section communicating AgentTailor's value proposition, feature highlights, pricing tiers (Free/Pro/Team), a call-to-action linking to signup, and a footer with navigation links.

## Context
The landing page is the front door to AgentTailor. It needs to instantly communicate what the product does ("makes AI agents dramatically better by assembling optimal context"), who it's for (developers and knowledge workers using ChatGPT and Claude), and why it matters (better context = better AI outputs). The pricing section is critical for conversion — users need to see a clear free tier to reduce signup friction, with paid tiers that feel like natural upgrades. The page must be fast (under 2 seconds) since it's the first impression and impacts SEO.

## Subtasks
1. Initialize `landing/package.json` with dependencies: React, Vite, Tailwind CSS, `@headlessui/react` for accessible components. Add scripts: `dev`, `build`, `preview`.
2. Create `landing/tailwind.config.ts` — Tailwind configuration:
   - Extend theme with AgentTailor brand colors (primary, secondary, accent).
   - Configure content paths for `src/**/*.{tsx,ts}`.
   - Add custom font family (Inter or similar clean sans-serif).
3. Create `landing/vite.config.ts` and `landing/index.html` — Vite entry point with meta tags for SEO (title, description, Open Graph).
4. Create `landing/src/main.tsx` — React entry point rendering the landing page.
5. Create `landing/src/components/Hero.tsx`:
   - Headline: concise value proposition (e.g., "Give your AI agents the context they need").
   - Subheadline: one sentence expanding on how AgentTailor works.
   - Primary CTA button: "Get Started Free" linking to signup.
   - Secondary CTA: "See how it works" scrolling to features section.
   - Optional: animated illustration or product screenshot placeholder.
6. Create `landing/src/components/Features.tsx`:
   - 4-5 feature cards highlighting key differentiators:
     - Cross-platform: works with ChatGPT (Custom GPT) and Claude (MCP Server + Extension).
     - Smart context assembly: automatically finds the most relevant docs and web results.
     - Transparency: see exactly what context was assembled and why (source attribution).
     - Quality scoring: know how well the context covers your task before using it.
     - Easy setup: upload docs, connect your AI, start getting better results.
   - Each card: icon placeholder, title, 1-2 sentence description.
7. Create `landing/src/components/Pricing.tsx`:
   - Three-column pricing grid: Free, Pro ($19/mo), Team ($49/mo).
   - Free: 50 requests/day, 3 projects, 20 docs per project, community support.
   - Pro: 500 requests/day, 20 projects, 100 docs per project, priority support, quality analytics.
   - Team: 2000 requests/day, 100 projects, 500 docs per project, dedicated support, team management, SSO.
   - Highlight Pro as "Most Popular".
   - Each tier has a CTA button linking to signup with plan preselected.
8. Create `landing/src/components/Footer.tsx`:
   - Navigation links: Features, Pricing, Documentation, Privacy Policy, Terms.
   - Copyright notice.
   - Social links placeholders (GitHub, Twitter/X).
9. Add `landing` to root `package.json` workspaces array.
10. Ensure the page is responsive: test layout at 320px, 768px, 1024px, 1440px breakpoints.

## Files to Create/Modify
- `landing/package.json` — Package definition with React, Vite, Tailwind
- `landing/tailwind.config.ts` — Tailwind configuration with brand theme
- `landing/vite.config.ts` — Vite configuration
- `landing/index.html` — HTML entry point with SEO meta tags
- `landing/src/main.tsx` — React entry point
- `landing/src/components/Hero.tsx` — Hero section with value proposition and CTAs
- `landing/src/components/Features.tsx` — Feature highlights grid
- `landing/src/components/Pricing.tsx` — Three-tier pricing comparison
- `landing/src/components/Footer.tsx` — Footer with navigation and legal links
- `package.json` — Add landing to workspaces (modify)

## Acceptance Criteria
- [ ] Landing page loads in under 2 seconds on simulated 3G connection
- [ ] Hero section clearly communicates what AgentTailor does and for whom
- [ ] "Get Started Free" CTA is prominently visible above the fold
- [ ] Features section highlights at least 4 key differentiators with descriptions
- [ ] Pricing section shows 3 tiers (Free, Pro, Team) with feature comparison
- [ ] Pro tier is visually highlighted as "Most Popular"
- [ ] Each pricing tier has a CTA button linking to signup
- [ ] Page is fully responsive at 320px, 768px, 1024px, and 1440px breakpoints
- [ ] Footer includes links to documentation, privacy policy, and terms
- [ ] HTML includes proper meta tags for SEO (title, description, Open Graph)
