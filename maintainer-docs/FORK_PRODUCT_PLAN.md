# OpenSEO fork: from personal workspace to customer product

Drafted 2026-09-26 for `janisbelozerovs-dev/open-seo` at `0ffff93`.

## Goal and starting point

First, make this fork reliable for one owner using real sites and real SEO data. After that, turn the proven workflow into a separate hosted product for customers. Keep these as two release gates: a private instance does not establish that customer signup, isolation, billing, and support are ready. The selected migration direction is recorded in [ADR: Vercel, Neon, and Serper](./ADR_VERCEL_NEON_SERPER.md).

The fork currently matches the upstream `every-app/open-seo` HEAD. It already contains the main application, not a starter template: keyword research, rank tracking, domain and competitor research, backlinks, site audits, AI visibility, reports, and an MCP server. The root app is TanStack Start on Cloudflare Workers with D1 by default; `web/` is a separate marketing site. The repository has unit tests and CI, but this fork's GitHub Actions page had zero runs when checked. Local checks and builds have since passed; a real deployment remains untested.

## Gate 1: personal working instance

**Chosen route:** port the working app to Vercel, use Neon Postgres, and make both a bounded manual SERP/rank-check workflow and a site audit work before the first preview. The owner has existing Serper credits for search results. The current Cloudflare self-host guide remains a reference for the original implementation, not the deployment instructions for this route. Docker is a useful local evaluation route, but the default Compose file pulls `ghcr.io/every-app/open-seo:latest` rather than an image from this fork. Docker uses `local_noauth` and does not run rank tracking schedules.

1. **Establish a reproducible baseline.** Use Node 22 or 24 and the pinned `pnpm@10.30.1`; install with the frozen lockfile. The local checks, tests, and builds passed on 2026-09-26, and the fork's first CI run passed both application and Docker jobs. Record the exact commit and deployment version. Keep upstream as a read-only remote and review incoming changes before merging them.
2. **Prepare the new runtime and data path.** Add a Vercel build target and a Neon connection using the existing Postgres schema and migrations. Add private session authentication; the current Cloudflare Access mode cannot protect Vercel, and `local_noauth` must remain local. Keep credentials in ignored local files and Vercel settings. Do not require DataForSEO for this path.
3. **Adapt both first workflows.** Use Serper for a bounded manual Google SERP/rank check, store snapshots in Neon, and clearly disable controls and metrics that Serper does not supply. Port the site audit's long-running crawl and scratch state to Vercel Workflow and Neon, starting with a bounded page count. The first audit uses the repository's `lighthouseStrategy: "none"` mode, so its page and link checks work without DataForSEO performance scores. Show expected query use before rank checks and account for failures and depleted credits. If eligible DataForSEO trial credit is available, use only a small, capped reference test to compare the existing behavior.
4. **Deploy privately and test a real domain.** After both workflows pass local checks, confirm the app build, sign-in gate, database migration, project creation, two rank checks, saved rank history, two site audits, audit issues, and provider-call count on a Vercel preview. Record broken steps, confusing copy, latency, and usage. Add Search Console, MCP, and reports only after their dependencies have been ported and tested.
5. **Harden the instance.** Document backup and restore for the chosen database and object store, an upgrade and rollback procedure, error monitoring, and a budget alert. Test a restore once. Review upstream support, pricing, documentation, analytics, and telemetry references before inviting anyone else; decide which should point to your service and which should be removed.

**Gate 1 passes when:** you can sign in, complete the workflow above without developer help, see no blocking setup errors, know the per-workflow data cost, and restore the project data from a backup.

## Gate 2: customer beta

This is a distinct deployment. The private Vercel account mode must not become public signup by default. The existing Cloudflare self-host setup gives everyone admitted by Access one shared workspace; the current `hosted` auth mode and organization code provide a starting point for customer accounts, but its billing and production infrastructure are tied to the upstream service. Do not deploy this fork's `hosted-prod` stage unchanged.

1. **Define the first customer and promise.** Choose one primary buyer and workflow, such as an independent SEO consultant tracking client sites. Interview 3–5 potential users and set a narrow beta scope. Decide whether customers bring their own data-provider key or buy included usage; that choice drives pricing, abuse protection, and billing.
2. **Separate your product identity and infrastructure.** Give this fork its own product name/domain, support address, documentation, GitHub links, analytics, OAuth callbacks, and deployment stage. Replace upstream billing, marketing, and referral references deliberately. Preserve the license and attribution notices. Deploy a staging environment and then a fresh production environment under your account; do not adopt upstream resource names.
3. **Make accounts safe for customers.** Validate signup, email verification, password recovery, organization membership and project isolation, invitation flow, token storage, account deletion, and rate limits. Test these across two unrelated customer accounts. The shared Cloudflare Access workspace from Gate 1 is not the multi-tenant product.
4. **Control cost and operations.** Implement and test per-customer usage caps, clear estimates before costly operations, billing or BYO-key behavior, provider failure handling, audit logs, backups, restore, monitoring, and incident response. Run the full CI and end-to-end smoke path for each release. Add a small staging data set that does not require expensive provider calls.
5. **Run a limited beta.** Invite 3–5 users, observe setup completion and repeat use, collect support issues, and fix the top blockers before broad signup. Publish pricing, privacy, terms, and support information appropriate to the customer deployment.

**Gate 2 passes when:** a new customer can sign up, connect data, complete the core workflow, understand charges, and get help; two customers cannot access each other's projects; a failed deploy can be rolled back without losing data; and support/costs are manageable for the beta cohort.

## Work order for the next iteration

1. Implement the Vercel runtime, Neon database connection, and private sign-in as the first reviewable slice.
2. Implement the bounded Serper rank-check path and the durable site-audit crawl, then deploy the private instance and run a real-domain smoke test.
3. Convert findings into a short prioritized issue list before expanding features.

## Migration assessment, 2026-09-26

The current source is not ready for a Vercel import. The root app's `vite.config.ts` uses the Cloudflare plugin and an auxiliary audit Worker. `wrangler.jsonc` binds D1, KV, R2, Durable Objects, Workflows, and scheduled triggers; Cloudflare Access protects private self-hosting. Vercel supports TanStack Start through Nitro, but moving this app there would require a new runtime adapter and replacements for those services, including durable audit/rank jobs, persistence, storage, and authentication. The separate `web/` marketing site is a smaller port, but its free-tool API uses Cloudflare Durable Objects and rate limits, so those endpoints also need replacements. Vercel hosting does not remove third-party SEO data costs. [Vercel's TanStack Start guide](https://vercel.com/kb/guide/deploy-a-tanstack-start-app-to-vercel) documents the Nitro route.

The repo's current data client covers SERPs, keyword ideas and volume, domain intelligence, backlinks, local business data, AI visibility, and Lighthouse through DataForSEO. A Serper integration could plausibly replace the Google search-results part, including a narrow rank check, after its response shape and location behavior are tested. It would not supply equivalent keyword volume, backlink, domain, AI visibility, or Lighthouse data; those features need other sources, reduced functionality, or clear unavailable states. The first implementation should add a provider boundary and adapt one workflow, not disguise Serper responses as complete DataForSEO data.

Current published entry costs do not favor Serper after the trial: [DataForSEO](https://dataforseo.com/pricing) lists a $50 minimum top-up, while [Serper](https://serper.dev/) advertises 2,500 free queries and a $50 first paid pack whose credits last six months. The owner has free Serper credits for a small private test. DataForSEO also documents a $1 new-account trial credit, which may allow a short evaluation but is not a lasting operating plan.

A low-cost initial workflow may instead be a project plus the built-in site crawler with Lighthouse turned off, followed by a report. Search Console can add real performance data for a verified site once Google OAuth is configured. This path still needs a code change: `scripts/selfhost-deploy-preflight.mjs` currently rejects a missing DataForSEO key, and provider-dependent actions must clearly report their unavailable state. It does not claim keyword volume, backlinks, or rankings without a source.

Cloudflare R2 activation is unconfirmed, but is no longer a gate for the selected Vercel route. Vercel object storage or another storage service will need its own cost check before site audits are ported.

## Current constraints and evidence

- GitHub Actions was enabled on this fork on 2026-09-26. The first [CI run](https://github.com/janisbelozerovs-dev/open-seo/actions/runs/36268672342) passed both the application and Docker image build jobs.
- Local baseline on Node 22.23.3 and pnpm 10.30.1: `ci:check` passed; 1,436 root tests and 33 website tests passed; the worker and website builds passed. The website's Miniflare test and prerender build needed localhost permission in this sandbox. No live SEO workflow or Cloudflare deployment was run.
- The machine defaults to Node 26 and pnpm 11; the pinned toolchain was used through temporary npm package binaries. Docker is not installed.
- No `.env.selfhost` file or local Alchemy login profile was present at the start of the Cloudflare setup work.
- No Neon project is linked to this checkout, the current shell has no Neon API key, and the Neon CLI is not installed. Vercel account linking remains to be checked when a preview build is ready.
- `docs/SELF_HOSTING_CLOUDFLARE.md` requires Cloudflare R2, Access, and DataForSEO. `docs/SELF_HOSTING_DOCKER.md` describes the local-only auth and rank scheduling limits.
- `compose.yaml` defaults to the upstream image; `.github/workflows/docker-image.yml` publishes only when the repository is `every-app/open-seo`.
- `alchemy.run.ts` and `docs/PREVIEW_DEPLOYMENTS.md` contain upstream production names and domains; `web/` contains upstream marketing and signup links.

## Decisions to make with the owner

- Which one or two SEO workflows must feel excellent in the first customer beta?
- Should future customers bring their own DataForSEO credentials, or will the product meter and bill for usage?
- Is the customer product intended to retain the OpenSEO name or have its own identity?
