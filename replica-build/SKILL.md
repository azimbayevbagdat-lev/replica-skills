---
name: "replica-build"
description: "Clone an app, step 4-7: build it screen by screen from the recon map, add auth/database/payments/email, test every flow for bugs, and score parity against the original."
---

# Replica, part 2: build, backend, test, diff

From the Replica pack (MIT), github.com/Jakeschincariol/replica-skill, pinned 77c9436. Reads `replica/recon.md`, `replica/architecture.md`, `replica/design/` written by replica-plan. If they are missing, run replica-plan first.

---

## Stage 4: build

Triggers: "build the clone", "build screen S07", "start building", "rebuild this screen". Updates `replica/features.csv` (`clone` column) and `replica/build-log.md`.

### Rules
- **Clean room.** Every line written fresh from the recon map and specs. Never paste the original's HTML, CSS, JS, SVGs or images, never load from its domain/CDN, never "view source and adapt".
- **Your words.** Every label, button, empty state and email written fresh. Matching what a button does is parity; matching its sentence is copying.
- **Tokens only.** No raw hex or pixel values in components.

### Steps
1. **Shell**: routing for every screen (stubs fine), layout, tokens, primitives, seed data. Commit.
2. **Vertical slice**: the core loop end to end before anything else. Ugly is fine. If backend is not ready, use a fake data layer with the same function signatures.
3. **Screen by screen** in architecture order: read its recon row; use the reference screenshot for layout and hierarchy, not pixels; build with primitives and real data; every state (empty, loading, filled, error, no permission, long content, mobile width); semantic HTML, labels, keyboard, visible focus, alt text; update features.csv; screenshot to `replica/clone-screens/S07.png` at the reference viewport; one commit per screen.

Done per screen: all states; works at 390px and 1440px; keyboard-only completes the flow; no console errors; no borrowed copy; features.csv updated; screenshot saved.

Build log: one line per screen (ID, date, done/partial, missing, surprises). If a feature is bigger than it looked, say so; never quietly ship half. When stuck, go back to the original as a user (help article, public walkthrough, own account), never its code or network calls.

---

## Stage 5: backend

Triggers: "add login", "set up auth", "wire up the database", "add payments", "connect Stripe", "send emails". Keeps `replica/backend.md`.

### Rules
- Official public APIs with the user's own keys only. Never the original's private endpoints or OAuth client.
- **The user creates accounts and keys.** Claude never signs up for services or types a password, card or live key. The user puts keys in `.env.local`; Claude writes `.env.example` with names, no values.
- Test mode first (Stripe test keys and cards).

### Parts
- **Auth**: email sign-up with verification, reset, magic link if the original has it, OAuth with the user's own apps; http-only secure cookies; sign out everywhere; roles via one "can this user do this" function; real account deletion (Apple requires it).
- **Database**: checked-in migrations run by a script; access rules on every table, tested with a second user getting nothing; seed with fake data (no real people); backups on.
- **Payments**: Stripe Checkout + Customer Portal, no custom card forms; webhooks verify signature, store event id, idempotent; handle checkout.session.completed, customer.subscription.updated/deleted, invoice.payment_failed; subscription state in your DB via webhooks; one-click cancel.
- **Email and jobs**: Resend/Postmark from own domain, fresh templates; jobs with retries and dead-letter log; UTC stored, local shown.
- **Integrations**: official API, fewest OAuth scopes, provider review (Google verification takes weeks; start early), rate limits.

Security checklist: secrets only in env, `.env*` gitignored; server-side validation (zod); authorisation on every read/write tested with a second user; rate limits on auth and anything sending email/SMS; webhook signatures; upload size/type limits on a separate bucket; no user data in URLs or logs; `npm audit`; privacy policy lists every processor.

---

## Stage 6: test

Triggers: "test my clone", "find bugs", "QA this", "write e2e tests". Writes `replica/test-plan.md`, `replica/bugs.md`, `e2e/`.

**Test your clone, not the original.** Never load-test, fuzz or script the original's servers.

1. **Plan** per flow: happy path; edge cases (empty, very long, emoji and accents, two tabs, double submit, back/refresh mid-flow, slow network, offline, expired session, second user's data invisible, time zones/DST, mobile, keyboard only, screen reader labels); negative cases (wrong password, declined card 4000 0000 0000 0002, permission denied, deleted record). Number cases F01-H1, F01-E3, F01-N2.
2. **Automate** with Playwright against local dev + seed data; selectors by role/label, never CSS classes; fail on console errors and any 5xx; axe scan (@axe-core/playwright) on each screen.
3. **Manual pass** for emails, real OAuth, payments end to end, visual glitches; drive the local clone in a browser if available, else give the user a checklist.
4. **Bugs**: ID, severity, exact steps, expected, actual, evidence. S1 data loss/security/payments/core flow blocked; S2 feature broken no workaround; S3 workaround or visibly wrong; S4 cosmetic. Only report what you reproduced; maybes go in "to check".
5. **Fix loop**: S1 and S2 first; failing test first, fix, keep the test, rerun suite; note fixing commit.

Ship nothing with an open S1.

---

## Stage 7: diff

Triggers: "how close is my clone", "what's missing", "parity check", "is it ready". Writes `replica/parity.md`.

**Parity means it does the same job, not that it looks the same.** Never chase pixel parity; the original's exact look is its trade dress.

1. **Feature parity** from features.csv (clone = yes / partial with note / no / skip with reason). Weights must 3, should 2, could 1; partial counts half; skip rows and rows the original lacks are excluded. Report score, must-haves done of total, weakest areas, missing list in build order. (Tool: `parity.py` in the repo.)
2. **Layout diff**: screenshots at the same viewport (1440x900 or 390x844) and same state. Compare structure via edge maps on a grid, ignoring colour (`imgdiff.py`): matches 90+, close 75+, partly 50+, different. Pixel mode only for your own regressions.
3. **Behaviour diff**: clicks to finish the core flow, error behaviour, what is remembered, emails sent. Write: flow, original does, clone does, fix or keep.
4. **Verdict**: not shippable (any must-have missing or open S1); shippable (all must-haves, feature score 80+, no open S1/S2); better than the original (shippable plus entrepreneur fixes, the goal). Honest numbers: 62% is 62%.

Output: parity.md, diff images, top five things to build next. Then replica-launch (entrepreneur stage) once parity is there.