# Drip: Launch Plan

Goal: get Drip into real users' hands as fast as possible without shipping something that destroys trust (bad tags, bad outfits).

Source of truth for scope: the PRD (`Drip: AI Personal Stylist PRD`). This plan sequences it for speed.

**Target:** usable by the founder in 4 weeks, private beta in 6 weeks, public launch in 8 to 10 weeks.

## Guiding rules

1. **Kill the biggest risk first.** If Workers AI cannot tag clothes (especially ethnic wear) accurately, nothing else matters. Test it on day 1, before building the app around it.
2. **Deploy from day 1.** Every merge to `main` deploys. No "big bang" release.
3. **Collect demand while building.** The landing page and waitlist go live in week 1, not at launch.
4. **Dogfood before beta.** The founder uses Drip daily for 2 weeks. If acceptance is under 60%, fix quality before adding users.
5. **Cut scope, not quality.** Anything not needed to answer "what do I wear today?" waits.

## Phase 0: Foundations (days 1 to 3)

| # | Task | Done when |
| --- | --- | --- |
| 0.1 | Scaffold monorepo (pnpm workspaces): `apps/app`, `apps/landing`, `packages/shared` | `pnpm install && pnpm build` passes |
| 0.2 | Add GitHub Actions deploy workflow: install, typecheck, test, `d1 migrations apply`, `wrangler deploy` on push to `main`. Uses the `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN` secrets | A push to `main` updates the live Worker |
| 0.3 | Check the API token has edit rights for Workers, D1, R2, Queues, Vectorize, Workers AI and AI Gateway | First deploy succeeds with no permission errors |
| 0.4 | Switch the Cloudflare account to Workers Paid. Create D1 db, private R2 bucket, Queue + dead-letter queue, Vectorize index, AI Gateway | All resources exist in `wrangler.jsonc` for `production` and `preview` |
| 0.5 | **Tagging spike:** label 100 real closet items (at least 25 ethnic). Run Workers AI vision with a strict JSON schema | Category + colour accuracy measured. Gate: 85% or better |
| 0.6 | **Multi-item spike:** test bounding boxes on 20 multi-item photos | Decide: support multi-item photos, or ask for one item per photo |
| 0.7 | Landing page v0 (Astro, static): headline, 3 benefit lines, waitlist form. Deploy to Vercel or GitHub Pages | Live URL collecting emails |

If 0.5 fails the gate: route tagging through AI Gateway to an external vision model and measure again. Do not continue until a model passes.

## Phase 1: Closet and capture (week 1 to 2)

| # | Task | PRD ref |
| --- | --- | --- |
| 1.1 | D1 migrations for the full data model in PRD section 8 | 8 |
| 1.2 | Device-linked auth: create closet, `drip_device` cookie, `requireDevice`, recovery key, rate limits | 9.1 |
| 1.3 | Bulk upload to R2 (up to 50 photos), one queue message per photo | FR-01 |
| 1.4 | Queue consumer: crop, background removal (Cloudflare Images), tag, embed into Vectorize | FR-02, FR-03 |
| 1.5 | Swipe review queue, low-confidence tags first | FR-04 |
| 1.6 | Closet grid with images streamed through the Worker | 9.1 |
| 1.7 | Style profile quiz | FR-06 |

**Exit:** the founder's full closet is catalogued in one evening.

## Phase 2: Dress me (week 2 to 4)

| # | Task | PRD ref |
| --- | --- | --- |
| 2.1 | Occasion parsing (presets + free text) with cached LLM parses | FR-08 |
| 2.2 | Weather lookup, cached per city and hour | FR-09 |
| 2.3 | Rule filter, candidate builder (around 200 candidates), rule scoring | 7 |
| 2.4 | LLM ranks top 15, returns 3 outfits with reasoning, styling notes, swaps | FR-10 |
| 2.5 | Flat-lay collage from cut-outs | FR-11 |
| 2.6 | Lock an item and build around it | FR-12 |
| 2.7 | Gap check with "your closet is thin here" note | 7 |
| 2.8 | Log every recommendation (inputs, scores, output) for debugging | 10 |

**Exit:** "Dress me" returns 3 outfits in under 8 seconds at p90.

## Phase 3: Learn and close the loop (week 4)

| # | Task | PRD ref |
| --- | --- | --- |
| 3.1 | Feedback actions with reason chips, updating learned weights | FR-14 |
| 3.2 | Wear log and calendar | FR-15 |
| 3.3 | Gap analysis and ranked buy list (top 5, outfits unlocked) | FR-20 |
| 3.4 | PWA: installable, offline closet, iOS standalone handling, "plan tomorrow" push | FR-23 |
| 3.5 | Data export (JSON + images zip) and full delete | 9.1, 10 |

**Exit:** MVP feature complete.

## Phase 4: Dogfood gate (week 4 to 6)

- Founder uses Drip every day for 2 weeks.
- Track the PRD metrics weekly: acceptance rate, time to decision, tagging correction rate.
- Fix the worst suggestions from the recommendation logs every 2 to 3 days.
- **Gate to beta:** acceptance of 60% or more, no tag error that is common across items, no data loss.

In parallel (low effort):
- Landing page v1: screenshots and a short screen recording of a real "Dress me" flow.
- Write the privacy policy and consent screen (India DPDP Act): what is stored, where, how to export and delete.

## Phase 5: Private beta (week 6 to 8)

- Invite 20 to 50 users from the waitlist, targeting men 20 to 35 in India, ideally with a wedding or event coming up (strongest pain, fastest feedback).
- Onboard the first 10 personally over a call. Watch where they drop off in closet capture.
- Add a feedback button in the app that sends a screenshot.
- Watch cost per request against the PRD targets (under Rs 2 per outfit, under Rs 1 per item).
- Ship P1 items only if beta users ask for them: repeat guard (FR-13), shop-check (FR-21), duplicate detection (FR-05).

**Gate to launch:** 40% of beta users still active in week 3, acceptance holds at 60%, cost per user is known.

## Phase 6: Public launch (week 8 to 10)

- Open sign-up from the landing page.
- Launch channels: Instagram/Reels showing "same closet, 10 outfits" before/afters, Product Hunt, Reddit (r/IndianFashionAddicts, r/malefashionadvice), wedding-season timing.
- Growth hook: a shareable "my closet in numbers" card (outfits possible, most versatile item). Cheap to build and spreads well.
- Run the monthly closet audit (FR-18, FR-19) once users have 60 days of wear data.

## Cut list (deliberately not in MVP)

Multi-event planner (FR-16), trip packing (FR-17), audit and retire advice (FR-18, FR-19), AI try-on preview (FR-22), user body photos (FR-07), passkeys, affiliate links, native apps.

## Decisions needed this week

| Decision | Recommendation | Why it matters now |
| --- | --- | --- |
| Personal tool or product for others? | Product. Plan for beta users from day 1 | Changes onboarding polish, privacy work and cost targets |
| Tagging model | Workers AI if it passes the 85% gate, otherwise an external model via AI Gateway | Blocks Phase 1 |
| Multi-item photos | Decide from spike 0.6 | Biggest lever on capture effort |
| Monthly AI budget for MVP | Set a hard cap in AI Gateway | Stops cost surprises in beta |
| Landing host | GitHub Pages (free, same repo, no extra account) | Needed in week 1 |

## Top risks to timeline

| Risk | Mitigation |
| --- | --- |
| Tagging accuracy fails, especially ethnic wear | Spike on day 1, external model fallback through AI Gateway |
| Closet upload fatigue kills onboarding | "Start with 20 items" mode, multi-item photos |
| One developer, 6 phases | Hold the cut list firmly; every new idea goes to post-launch |
| iOS PWA storage quirks | Handle standalone mode in Phase 3, test on a real iPhone early |
