# OpenSEO fork: from personal workspace to customer product

Drafted 2026-09-26 for `janisbelozerovs-dev/open-seo` at `0ffff93`.

## Goal and starting point

First, make this fork reliable for one owner using real sites and real SEO data. After that, turn the proven workflow into a separate hosted product for customers. Keep these as two release gates: a private instance does not establish that customer signup, isolation, billing, and support are ready.

The fork currently matches the upstream `every-app/open-seo` HEAD. It already contains the main application, not a starter template: keyword research, rank tracking, domain and competitor research, backlinks, site audits, AI visibility, reports, and an MCP server. The root app is TanStack Start on Cloudflare Workers with D1 by default; `web/` is a separate marketing site. The repository has unit tests and CI, but this fork's GitHub Actions page had zero runs when checked. Local checks and builds have since passed; a real deployment remains untested.

## Gate 1: personal working instance

**Reference route, pending a hosting decision:** deploy this source to a private Cloudflare self-host stage. The repository's Cloudflare guide provisions D1, KV, R2, Workers, and a Cloudflare Access gate. It also retains scheduled rank checks. The owner is also considering Vercel; the migration work is assessed below. Docker is a useful local evaluation route, but the default Compose file pulls `ghcr.io/every-app/open-seo:latest` rather than an image from this fork. Docker uses `local_noauth` and does not run rank tracking schedules.

1. **Establish a reproducible baseline.** Use Node 22 or 24 and the pinned `pnpm@10.30.1`; install with the frozen lockfile. The local checks, tests, and builds passed on 2026-09-26, and the fork's first CI run passed both application and Docker jobs. Record the exact commit and deployment version. Keep upstream as a read-only remote and review incoming changes before merging them.
2. **Choose an affordable data scope, then prepare accounts and secrets.** The current self-host deploy requires a DataForSEO credential; the owner does not want its $50 minimum top-up. Do not treat Serper as a drop-in replacement for the whole app. Pick the first workflow and the provider work it needs using the assessment below. For the existing Cloudflare route, confirm R2 is enabled and choose the real inboxes allowed through Cloudflare Access. Keep `.env.selfhost` out of Git. Add OpenRouter only when testing the in-app AI agent; Google OAuth is optional for Search Console and Analytics. Set a small test budget and monitor provider usage during live tests.
3. **Deploy a private stage from the fork.** On the Cloudflare route, follow `docs/SELF_HOSTING_CLOUDFLARE.md` using a stage and account owned by this project. Verify the Access challenge before sharing the URL. Check `/api/health`, logs, and database status after deployment. On a Vercel route, complete and test the runtime replacements below before deploying.
4. **Run one complete workflow on a real domain.** Create a project, run the chosen first workflow, and produce a report where that workflow supports one. Test Search Console and MCP only if they are part of your immediate workflow. Record broken steps, confusing copy, latency, and provider cost. Expand to keyword research, rank tracking, and domain/backlink data as their providers become available.
5. **Harden the instance.** Document backup and restore for the chosen database and object store, an upgrade and rollback procedure, error monitoring, and a budget alert. Test a restore once. Review upstream support, pricing, documentation, analytics, and telemetry references before inviting anyone else; decide which should point to your service and which should be removed.

**Gate 1 passes when:** you can sign in, complete the workflow above without developer help, see no blocking setup errors, know the per-workflow data cost, and restore the project data from a backup.

## Gate 2: customer beta

This is a distinct deployment. The self-host setup gives everyone admitted by Cloudflare Access one shared workspace. The `hosted` auth mode and organization code provide a starting point for customer accounts, but the current production infrastructure code is tied to the upstream `app.openseo.so` domains and existing resource names. Do not deploy this fork's `hosted-prod` stage unchanged.

1. **Define the first customer and promise.** Choose one primary buyer and workflow, such as an independent SEO consultant tracking and auditing client sites. Interview 3–5 potential users and set a narrow beta scope. Decide whether customers bring their own DataForSEO key or buy included usage; that choice drives pricing, abuse protection, and billing.
2. **Separate your product identity and infrastructure.** Give this fork its own product name/domain, support address, documentation, GitHub links, analytics, OAuth callbacks, and deployment stage. Replace upstream billing, marketing, and referral references deliberately. Preserve the license and attribution notices. Deploy a staging environment and then a fresh production environment under your account; do not adopt upstream resource names.
3. **Make accounts safe for customers.** Validate signup, email verification, password recovery, organization membership and project isolation, invitation flow, token storage, account deletion, and rate limits. Test these across two unrelated customer accounts. The shared Cloudflare Access workspace from Gate 1 is not the multi-tenant product.
4. **Control cost and operations.** Implement and test per-customer usage caps, clear estimates before costly operations, billing or BYO-key behavior, provider failure handling, audit logs, backups, restore, monitoring, and incident response. Run the full CI and end-to-end smoke path for each release. Add a small staging data set that does not require expensive provider calls.
5. **Run a limited beta.** Invite 3–5 users, observe setup completion and repeat use, collect support issues, and fix the top blockers before broad signup. Publish pricing, privacy, terms, and support information appropriate to the customer deployment.

**Gate 2 passes when:** a new customer can sign up, connect data, complete the core workflow, understand charges, and get help; two customers cannot access each other's projects; a failed deploy can be rolled back without losing data; and support/costs are manageable for the beta cohort.

## Work order for the next iteration

1. Decide whether the first private instance should keep the Cloudflare reference architecture or use a Vercel port, and choose its one essential SEO workflow.
2. Implement the required low-cost provider path, deploy the private instance, and run a real-domain smoke test.
3. Convert findings into a short prioritized issue list before expanding features.

## Hosting and data-provider decision, 2026-09-26

The current source is not ready for a Vercel import. The root app's `vite.config.ts` uses the Cloudflare plugin and an auxiliary audit Worker. `wrangler.jsonc` binds D1, KV, R2, Durable Objects, Workflows, and scheduled triggers; Cloudflare Access protects private self-hosting. Vercel supports TanStack Start through Nitro, but moving this app there would require a new runtime adapter and replacements for those services, including durable audit/rank jobs, persistence, storage, and authentication. The separate `web/` marketing site is a smaller port, but its free-tool API uses Cloudflare Durable Objects and rate limits, so those endpoints also need replacements. Vercel hosting does not remove third-party SEO data costs. [Vercel's TanStack Start guide](https://vercel.com/kb/guide/deploy-a-tanstack-start-app-to-vercel) documents the Nitro route.

The repo's current data client covers SERPs, keyword ideas and volume, domain intelligence, backlinks, local business data, AI visibility, and Lighthouse through DataForSEO. A Serper integration could plausibly replace the Google search-results part, including a narrow rank check, after its response shape and location behavior are tested. It would not supply equivalent keyword volume, backlink, domain, AI visibility, or Lighthouse data; those features need other sources, reduced functionality, or clear unavailable states. The first implementation should add a provider boundary and adapt one workflow, not disguise Serper responses as complete DataForSEO data.

Current published entry costs do not favor Serper after the trial: [DataForSEO](https://dataforseo.com/pricing) lists a $50 minimum top-up, while [Serper](https://serper.dev/) advertises 2,500 free queries and a $50 first paid pack whose credits last six months. Existing Serper credit could still make it useful for a small private test. DataForSEO also documents a $1 new-account trial credit, which may allow a short evaluation but is not a lasting operating plan.

A low-cost initial workflow may instead be a project plus the built-in site crawler with Lighthouse turned off, followed by a report. Search Console can add real performance data for a verified site once Google OAuth is configured. This path still needs a code change: `scripts/selfhost-deploy-preflight.mjs` currently rejects a missing DataForSEO key, and provider-dependent actions must clearly report their unavailable state. It does not claim keyword volume, backlinks, or rankings without a source.

For Cloudflare, the owner has an account but R2 activation is unconfirmed. [Cloudflare's R2 setup guide](https://developers.cloudflare.com/r2/get-started/) says to open **Storage & databases → R2 → Overview** in the account and complete its subscription checkout if prompted. The repo's Cloudflare self-host route requires R2; a Vercel port would use a different storage service and would need its own cost check.

## Current constraints and evidence

- GitHub Actions was enabled on this fork on 2026-09-26. The first [CI run](https://github.com/janisbelozerovs-dev/open-seo/actions/runs/36268672342) passed both the application and Docker image build jobs.
- Local baseline on Node 22.23.3 and pnpm 10.30.1: `ci:check` passed; 1,436 root tests and 33 website tests passed; the worker and website builds passed. The website's Miniflare test and prerender build needed localhost permission in this sandbox. No live SEO workflow or Cloudflare deployment was run.
- The machine defaults to Node 26 and pnpm 11; the pinned toolchain was used through temporary npm package binaries. Docker is not installed.
- No `.env.selfhost` file or local Alchemy login profile was present at the start of the Cloudflare setup work.
- `docs/SELF_HOSTING_CLOUDFLARE.md` requires Cloudflare R2, Access, and DataForSEO. `docs/SELF_HOSTING_DOCKER.md` describes the local-only auth and rank scheduling limits.
- `compose.yaml` defaults to the upstream image; `.github/workflows/docker-image.yml` publishes only when the repository is `every-app/open-seo`.
- `alchemy.run.ts` and `docs/PREVIEW_DEPLOYMENTS.md` contain upstream production names and domains; `web/` contains upstream marketing and signup links.

## Decisions to make with the owner

- Which one or two SEO workflows must feel excellent in the first customer beta?
- Should future customers bring their own DataForSEO credentials, or will the product meter and bill for usage?
- Is the customer product intended to retain the OpenSEO name or have its own identity?
