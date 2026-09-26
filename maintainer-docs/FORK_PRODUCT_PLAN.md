# OpenSEO fork: personal launch, then customer product

Updated 2026-09-26 for `janisbelozerovs-dev/open-seo`. This plan records the owner's choice to use the repository's Cloudflare deployment path for the first working instance. Vercel and Neon were considered, but porting this application there would add work before its existing audit and rank tracking workflows could run.

## Goal and starting point

First, run this fork privately for the owner on real sites. Then use the working instance to define and build a separate customer product. A successful private deployment does not establish that customer signup, account isolation, billing, and support are ready.

This repository already contains the main TanStack Start application, keyword and rank tracking, site audits, reports, and an MCP server. The app runs on Cloudflare Workers and uses D1, KV, R2, Durable Objects, Workflows, scheduled jobs, and Cloudflare Access in its documented self-host mode. `web/` is a separate marketing site. Docker is a local evaluation option; its default Compose file pulls the upstream image, uses `local_noauth`, and does not run rank schedules.

The fork's application checks, 1,436 root tests, 33 website tests, and both builds passed locally on Node 22.23.3 and pnpm 10.30.1. GitHub Actions is enabled; the [first CI run](https://github.com/janisbelozerovs-dev/open-seo/actions/runs/36268672342) passed its application and Docker jobs. No live deployment or paid-provider workflow has been validated yet.

## Gate 1: private working instance

**Selected route:** deploy the existing [Cloudflare self-host configuration](../docs/SELF_HOSTING_CLOUDFLARE.md) to the owner's Cloudflare account. Use an eligible DataForSEO $1 new-account credit for a small smoke test if available. Use the owner's Serper credits to develop a lower-cost Google results and rank-check path after the original behavior has been measured. Both saved-keyword rank checks and the site audit must work before this gate passes.

1. **Prepare the accounts.** Confirm or create a Cloudflare account. In its dashboard open **Storage & databases → R2 → Overview** and activate R2 if prompted. Cloudflare's [R2 setup guide](https://developers.cloudflare.com/r2/get-started/) says a subscription is required and includes free monthly usage; the repository's guide says activation requires a payment method. Create or use an eligible DataForSEO account, if the owner chooses the trial, and keep its Base64 API credential private. Choose a real inbox for Cloudflare Access sign-in; the GitHub `users.noreply.github.com` commit address is not an inbox.
2. **Prepare the checkout.** Use the pinned Node/pnpm versions and the frozen lockfile. Copy `.env.selfhost.example` to the ignored `.env.selfhost`, then set `DATAFORSEO_API_KEY` and `ACCESS_ALLOWED_EMAILS` locally. Do not place either value in Git or chat. Log in with `pnpm alchemy login`, granting `access:write`, and run `pnpm alchemy cloudflare bootstrap` as described in the self-host guide. Review the stage and account before provisioning.
3. **Deploy privately.** Run the repository's `pnpm deploy:selfhost --yes` command. This provisions the D1 database, KV namespaces, R2 bucket, Workers, migrations, and Cloudflare Access application. Record the deployed URL and commit. Confirm sign-in and `/api/health` before entering site data. No upstream `hosted-prod` resources should be deployed for this fork.
4. **Prove both workflows with small limits.** Create one project for a site the owner controls. Use a few keywords and one country; record the provider balance before and after two rank checks and confirm saved history. Run a small site audit twice with the UI's Lighthouse option turned off (`lighthouseStrategy: "none"`), then inspect page issues and internal links. This retains the crawl and its SEO checks while avoiding DataForSEO Lighthouse calls. Record any provider calls, job failures, latency, and confusing steps. Stop the trial test before its credit is exhausted.
5. **Make repeat use affordable.** Add a narrow Serper adapter for Google search results and manual saved-keyword rank checks, verify location and result-depth behavior on real responses, and preserve rank history. Show absent rankings explicitly. Add a per-run query cap and usage estimate. Do not pretend Serper supplies DataForSEO keyword volume, backlinks, domain intelligence, AI visibility, or Lighthouse data; leave those workflows on DataForSEO or show a clear unavailable state. Decide which remaining paid features are useful enough to fund.
6. **Harden the private instance.** Document backup and restore for D1 and R2, test a restore, and record the upgrade and rollback procedure. Add error monitoring and budget alerts before recurring or larger jobs. Review upstream support, analytics, telemetry, and marketing references before sharing the instance.

**Gate 1 passes when:** the owner can sign in, repeat both chosen workflows without developer help, see accurate persisted results, understand their provider usage, and restore project data. The $1 trial alone is a smoke-test budget, not a sustainable data plan.

## Gate 2: customer beta

The Cloudflare Access self-host mode admits named emails into one shared workspace. It is suitable for the owner's private instance, not a customer account system. The repository has a separate `hosted` auth mode and organization code, but its billing and production infrastructure are tied to the upstream service. Do not deploy this fork's `hosted-prod` stage unchanged.

1. Choose a first customer and narrow promise, such as an independent consultant tracking client sites. Interview 3–5 potential users. Decide whether customers bring their own provider credentials or pay for metered usage.
2. Give the fork its own product identity, domain, support address, documentation, analytics, OAuth callbacks, and deployment stage. Replace upstream billing, marketing, and referral links deliberately; preserve license and attribution notices. Use separate staging and production resources under the owner's account.
3. Validate signup, email verification and recovery, organization and project isolation, invitations, secret storage, deletion, and rate limits. Test with two unrelated accounts. Keep the private Access workspace separate.
4. Add per-customer provider limits, clear cost estimates, billing or bring-your-own-key behavior, backups, restore, monitoring, and release rollback. Test the full smoke path for each release.
5. Invite 3–5 beta users, observe setup and repeat use, fix the main blockers, then publish pricing, privacy, terms, and support information appropriate to the hosted service.

**Gate 2 passes when:** a new customer can sign up, complete the core workflow, understand charges, and get help; unrelated customers cannot access each other's projects; and a failed deployment can be rolled back without data loss.

## Immediate work order

1. Owner: confirm or create the Cloudflare account, activate R2, and obtain an eligible DataForSEO trial credential if desired. Keep payment and API details in the provider dashboards or the ignored local environment file.
2. Maintainer: prepare the local self-host settings and deployment; run the small private smoke test for sign-in, rank history, and the site audit once the accounts are ready.
3. Maintainer: implement and verify the bounded Serper rank-check path, then make a short prioritized list from real usage before expanding features.

## Why the Vercel/Neon port is deferred

The root `vite.config.ts` uses the Cloudflare plugin and an auxiliary audit Worker. `wrangler.jsonc` binds D1, KV, R2, Durable Objects, Workflows, and schedules. A Vercel deployment would need a new runtime adapter plus replacements for background jobs, audit scratch state, storage, database access, and private authentication. Neon could hold relational data but would not replace the other services. The separate `web/` marketing site is smaller, though its free-tool endpoints also use Cloudflare services. The owner's change to Cloudflare avoids that port for the first instance. Vercel and Neon can be revisited if the working product provides a concrete reason to move.

Serper is a plausible source for a narrow Google results workflow, not a drop-in replacement for the entire DataForSEO client. The repository's [DataForSEO setup guide](../docs/DATAFORSEO_API_KEY.md) documents $1 of new-account credit and a $50 minimum top-up. [Serper](https://serper.dev/) advertises free initial queries. Confirm current balance and pricing in each account before enabling repeat jobs.

## Open customer-product decisions

- Which one or two SEO workflows should the first customer beta do exceptionally well?
- Will customers provide their own data-provider credentials, or will the product meter and bill for usage?
- Should the customer product retain the OpenSEO name or have its own identity?
