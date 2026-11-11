---
name: "replica-plan"
description: "Clone an app, step 1-3: reverse-engineer it (screens, flows, data model, feature matrix), plan stack/schema/API, rebuild its design system as tokens. Use for 'clone this app', 'reverse engineer X'."
---

# Replica, part 1: recon, architect, design

From the Replica pack by Jake Schincariol (MIT), github.com/Jakeschincariol/replica-skill (pinned commit 77c9436). Pipeline: **replica-plan** (recon -> architect -> design) -> **replica-build** (build -> backend -> test -> diff) -> **replica-launch** (entrepreneur -> brand -> launch -> deploy). Every stage reads and writes files in a `replica/` folder in the user's project. Templates and the optional standard-library Python tools (contrast.py, parity.py, imgdiff.py, reviews.py, sweep.py, listing.py) live in that repo; if a tool is unavailable, do the same check by hand.

**It rebuilds what an app does, never what it owns.** Features and flows, clean-room style. Never its code, logo, copy, content or private APIs.

---

## Stage 1: recon

Triggers: "clone this app", "reverse engineer X", "how does X work", "what screens does X have", an app URL or store link to rebuild.

Output: `replica/recon.md` (recon map), `replica/features.csv` (columns: feature, area, priority, original, clone, notes), `replica/screens/` (reference screenshots, never shipped).

### Rules
- Public sources and the user's own account only. Never log into another account, never ask for a password, never get past a paywall or login.
- Reading, not scraping. No crawlers, bulk downloads or loops. Browser at human speed with the user present.
- No source code, no private APIs: do not read/save JS bundles, decompile binaries or log network calls to copy endpoints. Public API docs are fine.
- Check the terms. If the user's account terms forbid building a competitor, say so and use public sources only.
- Screenshots are reference only.

### Steps
1. **Scope** (ask or propose and get a yes): which app and platform; which slice (default: the core loop users pay for); who it is for.
2. **Sources table**, URL on every row: help center/docs, pricing page, changelog, store listing, public walkthrough videos, marketing site, user's own account, public API docs.
3. **Screen inventory**: `ID | screen | route | purpose | key components | states seen` with stable IDs S01, S02... Record states: empty, loading, filled, error, permission denied, mobile.
4. **User flows** F01, F02...: goal, screens passed through, edge cases, click count on happy path (the number to beat).
5. **Components**: name, variants, states, screens used.
6. **Inferred data model**: entities, fields, relationships, each with evidence and confidence. Mark guesses as guesses.
7. **Feature matrix**: priority must/should/could; `clone` starts `no`.
8. **What cannot be cloned**: `skip` rows with reason (licensed content, the network and its users, owned data, partner deals, hardware, regulated licences).
9. **Size it**: S (weekend), M (weeks), L (quarter), XL (rescope). No promises of a perfect clone.

Summary: core loop, screen and flow counts, three hardest parts, out of scope, next step.

---

## Stage 2: architect

Triggers: "plan the clone", "what stack", "design the database", "write the schema", "plan the API". Needs `replica/recon.md`; if missing, run recon first. Writes `replica/architecture.md` and `replica/schema.sql`.

1. **Stack**: use what the user knows; otherwise default: Next.js (App Router) + TypeScript, Tailwind, Expo for mobile, Postgres on Supabase or Neon, Drizzle or Prisma, Supabase Auth or Auth.js, Stripe Checkout + Billing, Resend or Postmark, Vercel Cron/Inngest/Trigger.dev, Supabase Storage or R2, Vercel hosting. One line of why each. One database, no microservices.
2. **Schema**: every table gets `id uuid primary key default gen_random_uuid()`, created_at, updated_at; owner column (user_id/org_id); FKs with a decided `on delete`; indexes on every FK and filtered/sorted column; enums or check constraints for status; `timestamptz` in UTC; money as integer cents + currency; access rules (RLS on Supabase or one authorisation check per query). Hard constraints (e.g. double-booking) belong in the database (exclusion constraint/unique index).
3. **API**: per flow, `method path | what | who can call | input | output | flow`, plus webhooks in/out and background jobs with schedules. Official public APIs with the user's own keys only.
4. **Parts that bite**: time zones/DST, idempotency, races, rate limits, file limits, search, realtime, offline, email deliverability, multi-tenancy, GDPR deletion.
5. **Build order**: vertical slice of the core loop first, then must-haves, should-haves, could-haves, then entrepreneur fixes. Each milestone lists screens, tables, routes.

Summary: stack in one line, table count, route count, three riskiest parts.

---

## Stage 3: design

Triggers: "match the design", "rebuild the design system", "colours and fonts", "design tokens". Writes `replica/design/tokens.json`, `tokens.css`, Tailwind mapping, `replica/design/components.md`.

### Rules
Rebuild the system (roles, scales, patterns), never: logos, icons, illustrations, photos, sounds (use Lucide/Phosphor/Heroicons/Tabler, own illustrations); licensed fonts (use Inter, Geist, IBM Plex, Manrope, Source Serif); their copy; their brand colour (record as role `accent` with a neutral placeholder; replica-brand replaces it).

### Steps
1. **Measure, do not guess** from screenshots: colour roles (bg, surface, border, border-input, text, text-muted, accent, on-accent, danger, success, warning; usually 5-7 greys), type scale (e.g. 12/14/16/20/28/40), spacing (4 or 8 base), 2-3 radii/shadows/motion, layout (max width, grid, breakpoints, sidebar, header).
2. **Tokens**: fill tokens.json with role names, generate tokens.css custom properties, map into Tailwind so components use `bg-surface text-muted`, never raw hex. Add a `pairs` list and check WCAG: AA 4.5:1 body text, 3:1 large text, input borders and focus rings. Fix in tokens.
3. **Component specs**, one block each: variants, sizes, states (default, hover, active, focus-visible, disabled, loading, error, empty), tokens, a11y, screens used.
4. **Build primitives** once in isolation (a /design route or Storybook) on an accessible base (Radix, shadcn/ui, React Aria). Screenshot it.

Output: tokens, Tailwind config, components.md, primitives, zero AA failures. Next: replica-build.