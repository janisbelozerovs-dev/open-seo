# ADR: Vercel, Neon, and Serper for the fork's first product

**Status:** Accepted direction; implementation pending  
**Date:** 2026-09-26  
**Decider:** Fork owner

## Context

The owner wants to use the fork privately before offering it to customers. They prefer Vercel for hosting, Neon for Postgres, and existing free Serper credits for search data. They do not want to make DataForSEO's $50 minimum top-up to get started.

This is a port of the application, not a hosting setting. The root TanStack Start app is built with `@cloudflare/vite-plugin`; its runtime uses Cloudflare D1, Hyperdrive, KV, R2, Durable Objects, Workflows, cron, and Access. The `web/` marketing site is separate and has Cloudflare-backed free-tool endpoints. Vercel supports TanStack Start with Nitro, and its Workflow SDK supports TanStack Start, but neither automatically translates Cloudflare bindings. [Vercel deployment guide](https://vercel.com/kb/guide/deploy-a-tanstack-start-app-to-vercel); [Vercel Workflow support](https://vercel.com/changelog/workflow-sdk-now-supports-tanstack-start).

The repository already has a Postgres schema and migrations in `src/db/pg/` and `drizzle-pg/`. Its runtime connection currently insists on a Cloudflare Hyperdrive binding in `src/db/provider.ts`, so Neon needs a Node runtime connection path. The repo's DataForSEO client also provides more than Google results: keywords, domain and backlink data, local business data, AI visibility, and Lighthouse. Serper is a Google results source, not an equivalent replacement for all those datasets.

## Decision

1. Target the **working OpenSEO app** on Vercel and a Neon Postgres database. Keep the marketing site out of the first deployment; adapt it later if needed.
2. Make the first private workflow **manual Google SERP and rank checks for a small set of saved keywords** using the owner's Serper credits. Start with one country and desktop results; validate location, result depth, and domain matching against real responses before offering more controls. Store results and history in Neon. Show an explicit “outside checked results” state when the target is absent.
3. Keep DataForSEO-backed capabilities unavailable or clearly labelled until each has a real replacement. Never turn a Serper result into invented search volume, backlink counts, difficulty, or AI visibility metrics. Do not require `DATAFORSEO_API_KEY` for this new deployment path.
   If the owner has an eligible $1 DataForSEO trial credit, use a small amount for a reference test of the original endpoints and comparison with Serper; do not make the live workflow depend on replenishing it.
4. Reuse the existing Postgres schema and Drizzle migrations. Use a pooled Neon URL for app requests and a direct URL for migrations, per [Neon's connection guidance](https://neon.com/docs/connect/connection-pooling). Preserve the Cloudflare path in the fork until the Vercel version passes its own checks.
5. Use a private account mode for the owner with real session authentication. The existing `cloudflare_access` mode depends on Access; the existing `hosted` mode couples identity with hosted billing. Separate private Vercel authentication from hosted billing rather than using `local_noauth` on an internet-facing app.
6. Add storage and long-running jobs only as each workflow requires them. The first manual rank check should avoid R2, KV, Durable Objects, and scheduled jobs. A later site-audit port needs durable jobs and object storage; Vercel Workflow and private Vercel Blob are candidates to validate against the current audit behavior. [Vercel Workflow support](https://vercel.com/changelog/workflow-sdk-now-supports-tanstack-start); [Vercel Blob](https://vercel.com/docs/vercel-blob).

## Why this sequence

The smallest useful slice proves the new runtime, database, authentication, provider connection, stored results, and real user workflow without first rebuilding every Cloudflare service. It also gives a measurable Serper query cost per rank check. Existing free credits make that test possible before a top-up. [Serper's current pricing page](https://serper.dev/) advertises 2,500 free queries; its first paid pack is $50 and those credits last six months. [DataForSEO](https://dataforseo.com/pricing) also lists a $50 minimum top-up. A provider swap alone therefore does not solve the later paid entry cost.

## Implementation slices and acceptance checks

1. **Vercel runtime:** add a separate Nitro build target, keep the Cloudflare build working, and serve a basic authenticated app route on a Vercel preview. The build must not import `cloudflare:workers` at runtime on that route.
2. **Neon data:** connect the app through a server-only pooled URL; run the committed Postgres migrations with a direct URL on a disposable Neon branch; create and read a project. Never expose either URL in client assets or logs.
3. **Private identity:** sign in as the owner, reject a signed-out request, persist sessions in Neon, and confirm project data is not available without a session. Customer signup and billing stay disabled for this gate.
4. **Serper rank check:** make a bounded request, validate the response, match the target domain, store a snapshot, and show it in the existing rank history UI. Check error and exhausted-credit states. Disable unsupported device, depth, location, and keyword-metric controls until verified or adapted.
5. **Cost guard:** cap keywords and provider calls per manual run, show the expected query count before starting, and record actual calls. No automatic schedule in the first gate.
6. **Private launch:** deploy a preview, run the workflow twice on a real domain, verify history and a restart, and record usage and failures. Keep the public customer route closed.

## Later work

- Port the crawler and audit orchestration to a durable job system, then replace its R2 payload store. Restore non-Lighthouse audits first; select a separate Lighthouse source if that feature is needed.
- Add a cache/storage replacement for R2 and KV-dependent features, and port MCP/OAuth and the in-app chat only when those are part of the chosen product promise.
- Research separate sources for keyword volume, backlinks, domain intelligence, local business data, and AI visibility; price and test each before enabling it.
- When inviting customers, validate account isolation, billing, provider usage caps, backups, support, and the marketing site. [Vercel's Hobby plan](https://vercel.com/docs/plans/hobby) is for non-commercial personal use; the customer product needs a commercial plan.

## Open items before a live deployment

- A Vercel account and a Neon project are not linked to this checkout. The owner should connect them when the first preview build is ready; credentials belong in Vercel/Neon settings, not Git or chat.
- Confirm remaining Serper credits and the exact first domain, country, and keywords for the private smoke test. Do not paste the Serper API key into chat.
- If DataForSEO trial credit is available, record its starting balance and cap reference calls so the test stays within that credit.
- Verify Vercel and Neon usage limits for the selected accounts before enabling recurring jobs or customer traffic.
