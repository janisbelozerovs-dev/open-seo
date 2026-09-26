# OpenSEO fork: from personal workspace to customer product

Drafted 2026-09-26 for `janisbelozerovs-dev/open-seo` at `0ffff93`.

## Goal and starting point

First, make this fork reliable for one owner using real sites and real SEO data. After that, turn the proven workflow into a separate hosted product for customers. Keep these as two release gates: a private instance does not establish that customer signup, isolation, billing, and support are ready.

The fork currently matches the upstream `every-app/open-seo` HEAD. It already contains the main application, not a starter template: keyword research, rank tracking, domain and competitor research, backlinks, site audits, AI visibility, reports, and an MCP server. The root app is TanStack Start on Cloudflare Workers with D1 by default; `web/` is a separate marketing site. The repository has unit tests and CI, but this fork's GitHub Actions page had zero runs when checked. Local checks and builds have since passed; a real deployment remains untested.

## Gate 1: personal working instance

**Chosen route:** deploy this source to a private Cloudflare self-host stage. The repository's Cloudflare guide provisions D1, KV, R2, Workers, and a Cloudflare Access gate. It also retains scheduled rank checks. Docker is a useful local evaluation route, but the default Compose file pulls `ghcr.io/every-app/open-seo:latest` rather than an image from this fork. Docker uses `local_noauth` and does not run rank tracking schedules.

1. **Establish a reproducible baseline.** Use Node 22 or 24 and the pinned `pnpm@10.30.1`; install with the frozen lockfile. The local checks, tests, and builds passed on 2026-09-26. Obtain a green CI run in this fork before changing features. Record the exact commit and deployment version. Keep upstream as a read-only remote and review incoming changes before merging them.
2. **Prepare accounts and secrets.** Use a Cloudflare account with R2 enabled, a DataForSEO account and credential, and the exact email addresses allowed through Cloudflare Access. Keep `.env.selfhost` out of Git. Add OpenRouter only when testing the in-app AI agent; Google OAuth is optional for Search Console and Analytics. Set a small test budget and monitor the DataForSEO balance during live tests.
3. **Deploy a private stage from the fork.** Follow `docs/SELF_HOSTING_CLOUDFLARE.md` using a stage and Cloudflare account owned by this project. Verify the Access challenge before sharing the URL. Check `/api/health`, logs, and the database status after deployment.
4. **Run one complete workflow on a real domain.** Create a project; research and save keywords; run a site audit; add and check a small rank tracking set twice; inspect domain/backlink data; produce a report. Test Search Console and MCP only if they are part of your immediate workflow. Record broken steps, confusing copy, latency, and the DataForSEO cost of each workflow.
5. **Harden the instance.** Document D1 and R2 backup and restore, an upgrade and rollback procedure, error monitoring, and a budget alert. Test a restore once. Review upstream support, pricing, documentation, analytics, and telemetry references before inviting anyone else; decide which should point to your service and which should be removed.

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

1. Get a green GitHub Actions CI run on this fork. The equivalent local checks have passed.
2. Provision the private Cloudflare deployment and run the real-domain smoke test.
3. Convert findings into a short prioritized issue list before changing features.

## Current constraints and evidence

- GitHub Actions was enabled on this fork on 2026-09-26. It had no runs before that change; the draft PR's CI still needs verification.
- Local baseline on Node 22.23.3 and pnpm 10.30.1: `ci:check` passed; 1,436 root tests and 33 website tests passed; the worker and website builds passed. The website's Miniflare test and prerender build needed localhost permission in this sandbox. No live SEO workflow or Cloudflare deployment was run.
- The machine defaults to Node 26 and pnpm 11; the pinned toolchain was used through temporary npm package binaries. Docker is not installed.
- `docs/SELF_HOSTING_CLOUDFLARE.md` requires Cloudflare R2, Access, and DataForSEO. `docs/SELF_HOSTING_DOCKER.md` describes the local-only auth and rank scheduling limits.
- `compose.yaml` defaults to the upstream image; `.github/workflows/docker-image.yml` publishes only when the repository is `every-app/open-seo`.
- `alchemy.run.ts` and `docs/PREVIEW_DEPLOYMENTS.md` contain upstream production names and domains; `web/` contains upstream marketing and signup links.

## Decisions to make with the owner

- Which one or two SEO workflows must feel excellent in the first customer beta?
- Should future customers bring their own DataForSEO credentials, or will the product meter and bill for usage?
- Is the customer product intended to retain the OpenSEO name or have its own identity?
