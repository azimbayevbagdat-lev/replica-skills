---
name: "replica-launch"
description: "Clone an app, step 8-11: mine real user reviews for fixes and a positioning angle, rebrand it as your own, write landing/pricing/store listing, deploy to your domain."
---

# Replica, part 3: entrepreneur, brand, launch, deploy

From the Replica pack (MIT), github.com/Jakeschincariol/replica-skill, pinned 77c9436. The entrepreneur stage can also run alone on an app the user is only thinking of cloning.

---

## Stage 8: entrepreneur

Triggers: "what do people hate about X", "read the reviews", "find the gap", "how do I position this", "make it sellable".

A straight copy has no reason to exist. Find what the original's users hate, in their words, and fix it.

### Rules (not negotiable)
- **Never fabricate** reviews, quotes, ratings, counts or sources. If a source is unreachable, say so. 14 reviews means say 14.
- **Every quote verbatim and linked.** Drop any row without a URL.
- **Reading, not scraping.** Read pages like a person. Official feeds are fine within terms: Apple RSS `https://itunes.apple.com/us/rss/customerreviews/id=<APP_ID>/sortBy=mostRecent/json`, HN Algolia `hn.algolia.com/api/v1/search?query=...`, Reddit's official API.
- **No fake reviews, ever**, for or against anyone.
- **Reviewers are not testimonials.**

### Steps
1. **Collect** 100+ reviews from 3+ sources, recent first: App Store, Google Play, G2/Capterra/Trustpilot (include 1-3 stars), Reddit ("X alternative", "switched from X", "X vs"), Hacker News, the original's public feature board with votes, its changelog (so you do not "fix" what is fixed). `replica/reviews.csv`: source,url,date,rating,text. Read 3-4 stars too.
2. **Rank** into themes, weighting low ratings and recent reviews; themes with under 3 reviews or one source are thin; list requests in users' words; read unmatched low ratings by hand.
3. **Three lists** (problem, review count, source count, 1-2 linked quotes): what they hate; what is missing; what is unsolved (ignored jobs or groups, which become positioning).
4. **Fix plan**: top 5-8 by evidence x cheapness; what to build, size S/M/L, which stage, evidence; add each to features.csv with `original` = no. Pricing complaints go to the launch stage.
5. **Angle**, three options: `For {who} who {hate this about the original}, {your app} {does this instead}. Evidence: {theme}, {n} reviews across {n} sources.` Recommend one. Never the original's name in your name, ads or listing.

Output: reviews.csv, feedback.md, fixes.md, new feature rows, summary stating the sample size.

---

## Stage 9: brand

Triggers: "name my app", "rebrand the clone", "make it mine", "logo brief", "brand voice". Always before launch. Writes `replica/brand.md` and `replica/brand.json` (`avoid`, `domains`, `colors`).

1. **Name**: 20 candidates across styles (descriptive, compound, invented, metaphor, verb), cut to 5: not confusingly similar to the original or same-category apps in sound, look or meaning (no puns on their name); short, spellable, no bad meaning in big languages; fits the angle.
2. **Checks, run not assumed**, each marked "to run" or result + date: US trademark (tmsearch.uspto.gov, classes 9 and 42), EU (EUIPO eSearch/TMview), Canada, WIPO Global Brand Database, domain whois, App Store and Play exact search, handles (X, Instagram, TikTok, GitHub), web search "name + category". Screening only; a trademark lawyer before spending money.
3. **Palette**: new primary hue from a different family than the original's, into the same token roles; zero AA contrast failures; put the original's colours in brand.json.
4. **Logo brief** (not a logo): idea in one line, mark type, works at 16px and 1024px, deliverables (SVG, 1024 iOS icon no transparency, favicon set, 1200x630 social image), must not resemble the original's mark.
5. **Voice**: three words with what each does not mean, five do/don't pairs, rewrite the 10 most-seen strings fresh.
6. **Sweep**: search contents and file names for the original's name (also inside identifiers), domains and colours, skipping node_modules, build output and replica/ (tool: `sweep.py`). Fix until clean. Check by eye: favicon, titles, email templates, OG image, app icon.

---

## Stage 10: launch

Triggers: "landing page", "pricing", "how much should I charge", "App Store listing", "launch plan", "Product Hunt". Writes `replica/launch/`: landing.md, pricing.md, listing.json, launch-plan.md.

Rules: no fake proof (testimonials, user counts, ratings, press logos, "trusted by"); no reviewer quotes as testimonials; the original's name stays out of name, listing, keywords, ads.

1. **Landing page**: hero (the angle, one line, one button, real product screenshot); the problem (top two hate themes, paraphrased); how it works in three steps; features led by the fixes; pricing; FAQ with real objections; final CTA. Brand voice, contrast checked.
2. **Pricing**: table of the original's and 2-3 alternatives' public pricing with URL and date read; reviewer price/billing complaints with counts; model (free/trial, seat vs flat, annual ~2 months free); max three tiers named by who they serve; fix billing complaints (one-click cancel, renewal emails, no surprise seat jumps); Stripe products for the user to create.
3. **Store listing**: limits App Store name 30, subtitle 30, promo 170, keywords 100, description 4000; Play title 30, short 80, full 4000. Flag ranking/price claims in short fields, emoji, wasted keyword characters, any `avoid` name (tool: `listing.py`). Apple Guideline 4.1 Copycats: make fixes and own brand visible in screenshots and first lines. Also: screenshots at current sizes, privacy labels/data safety, privacy and support URLs, age rating, reviewer demo account.
4. **Launch plan**: waitlist/beta, analytics and error tracking, where unhappy users of the original talk, a Product Hunt or HN post leading with the fix, the first 10 users to talk to by hand.

---

## Stage 11: deploy

Triggers: "deploy it", "ship it", "put it live", "connect my domain", "go to production", "publish the app". Writes `replica/deploy.md`.

Rules: **nothing goes live without the user's go**; **the user buys and signs in** (Claude never buys a domain, enters a card, types a password or pastes a live key; it writes the exact records, env var names and commands); **not until rebranded** (sweep clean, no exceptions).

1. **Preflight**, results pasted into deploy.md: Playwright tests green, parity must-haves done, sweep clean, listing lint passes, `npm run build` passes; by hand: no open S1/S2, privacy policy and terms live listing every processor, cookie banner if needed (EU/UK), account deletion works, favicon/titles/OG image are yours. Any failure stops the deploy.
2. **Production services**: separate production DB project with backups, migrations run by deploy; production env vars in the host; Stripe live mode (recreate products, prod webhook + secret, one real purchase then refund); OAuth redirect URIs and origins for the prod domain (Google verification approved); sending domain verified.
3. **Host and domain**: default Vercel (or Netlify, Cloudflare Pages, Fly, Render), repo connected, main deploys, PR previews. DNS: A @ -> host apex IP; CNAME www -> host target (Vercel cname.vercel-dns.com); TXT verification if asked. Email DNS: SPF, DKIM, DMARC starting `v=DMARC1; p=none; rua=mailto:you@yourdomain`, later quarantine. One canonical host, redirect the other; check HTTPS.
4. **Watch it**: error tracking, uptime checks on home and core-flow API, logs, privacy-friendly analytics, alerts; do the core flow live yourself and have the user do it on their phone.
5. **Mobile**: Expo `eas build` then `eas submit` to TestFlight and Play internal testing (or Xcode/Gradle). The user owns developer accounts ($99/yr Apple, $25 once Google). Beta first, then review.

Output: deploy.md with every check, live URL, DNS records, what to watch in week one. Not legal advice: check trademarks and terms, and see a lawyer before selling if money is on the line.