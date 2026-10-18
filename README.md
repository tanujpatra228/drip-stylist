# Drip

**An AI personal stylist that builds outfits from the clothes you already own.**

Tell Drip where you are going. It returns three outfits from your own closet in under a minute, explains why each one works, and only suggests buying something when your closet genuinely cannot cover the occasion.

Built as an installable PWA running entirely on Cloudflare.

## What it does

- **Digitise your closet.** Bulk photo upload, background removal, multi-item splitting and auto-tagging, with a swipe queue to fix wrong tags.
- **Dress me.** Occasion and weather aware outfits (office to sangeet) with styling notes and per-slot swaps.
- **Learns from you.** Every accept, swap and reject tunes your personal preferences.
- **Plan ahead.** Multi-event planning with no repeats, trip packing lists.
- **Closet audit.** Keep, restyle, retire suggestions backed by wear data, plus a ranked buy list showing how many new outfits each item unlocks.

## Principles

Closet first. Evidence over opinion. Explain every pick. Low effort capture. No upsell bias.

## Repository layout

Monorepo with two independently deployed apps:

```
drip-stylist/
├── apps/
│   ├── app/              # The product: Cloudflare Worker (API + PWA assets)
│   │   ├── worker/       # Hono API: auth, routes per module, queue consumer, crons
│   │   ├── web/          # React + Vite PWA (Tailwind, Workbox, IndexedDB)
│   │   ├── migrations/   # D1 SQL migrations
│   │   └── wrangler.jsonc
│   └── landing/          # Marketing site: static, deployed to Vercel or GitHub Pages
├── packages/
│   └── shared/           # Shared types and constants (tags, occasions, schemas)
└── docs/                 # PRD, architecture notes, decisions
```

`apps/app` and `apps/landing` share no runtime and deploy separately. The landing page is planned but intentionally minimal for now.

## Tech stack

| Layer | Choice |
| --- | --- |
| Runtime | Cloudflare Workers (TypeScript, Hono) |
| Frontend | React, Vite, Tailwind, Workbox PWA, served from the same Worker |
| Database | D1 (SQLite), every table scoped by `closet_id` |
| Storage | R2 (private bucket, images streamed via the Worker) |
| Async | Queues (one message per photo, with dead-letter queue) |
| AI | Workers AI via AI Gateway (vision tagging, ranking, explanations) |
| Search | Vectorize (item embeddings) |
| Images | Cloudflare Images (background removal) |
| Scheduling | Cron Triggers (cleanup, "plan tomorrow" push reminders) |
| Landing | Static site on Vercel or GitHub Pages |

**Auth:** no accounts or passwords. Each closet is opened from linked devices (hashed httpOnly cookie tokens), with link codes, QR linking and a one-time recovery key.

**AI design:** rules generate and score outfit candidates from owned items; the LLM only ranks the top 15 and writes the explanation. Cheap, consistent and grounded.

## Status

Pre-MVP. Planning and scaffolding. MVP target: 8 weeks, single user, built so it can open to more users without a rewrite.

## License

TBD
