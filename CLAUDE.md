# CLAUDE.md — plaidcloud-status

Setup-continuity notes for this repository. Read this first before changing anything.

## What this is

The public PlaidCloud status page, built on [Upptime](https://upptime.js.org): a GitHub-native
uptime monitor + status page + incident tracker. It runs **entirely on GitHub** (Actions, Issues,
Pages) and is **deliberately isolated from PlaidCloud infrastructure** so it stays up during a
PlaidCloud outage. Do not couple it to PlaidCloud's cloud/k8s in any way.

- **Live page (target):** https://status.plaidcloud.com
- **Repo:** https://github.com/PlaidCloud/plaidcloud-status (public)
- **Components monitored:** see [.upptimerc.yml](./.upptimerc.yml)
- **Incident/maintenance workflow:** see [RUNBOOK.md](./RUNBOOK.md)

## What we monitor

Three public websites, checked with plain HTTP GET (2xx = up). No custom health endpoints required.

| Component | Slug | URL |
| --- | --- | --- |
| Web App | `app` | https://app.plaidcloud.com/client/ (root `/` 302s here) |
| Documentation | `docs` | https://docs.plaidcloud.com/ |
| Website | `website` | https://plaidcloud.com/ |

## Branding

The status page is themed to match **plaidcloud.com** (palette sourced from the
`plaidcloud-website` repo, `src/components/CustomStyles.astro`):

- **Palette:** orange `#FF7514`, amber `#FDC20D`, navy `#041E4B`, ink `#101828`, cyan `#2EBDF1`, paper `#F8FAFD`; brand gradient orange→amber.
- **Font:** Inter (loaded from Google Fonts via `status-website.links`; degrades to system sans if unavailable).
- **Look:** light page, brand-navy navbar with the white PlaidCloud logo, orange links.

All theming lives in `.upptimerc.yml` under `status-website`:
- `theme: light` is the base (also supplies the semantic up/down/degraded status colors, which we intentionally leave alone).
- `css:` overrides the chrome CSS variables (`--body-*`, `--card-*`, `--nav-*`, `--submit-button-*`) with `!important` so they win regardless of cascade order, plus link/nav/selection styling. The full themeable variable list is in the built `global.css` on the `gh-pages` branch.
- `logoUrl` and `faviconSvg` are **embedded as data URIs** (white logo PNG + favicon SVG) so the page has no external image dependency — important for isolation. Source files: `plaidcloud-website/public/wp-content/uploads/2021/12/PlaidCloudLogo_White.png` and `plaidcloud-website/src/assets/favicons/favicon.svg`.

To re-theme: edit the `css` block / data URIs in `.upptimerc.yml` and push — Setup CI rebuilds and redeploys. Built-in `theme` options: `light`, `dark`, `night`, `ocean`.

## Current state

✅ **LIVE** at https://status.plaidcloud.com — valid TLS, all three sites monitored and green.
✅ Engine from the `upptime/upptime` template (v1.41.x); `GH_PAT` secret set; Pages serving from `gh-pages` with the custom domain; branded to plaidcloud.com (see Branding above).
✅ DNS: explicit `status` CNAME → `plaidcloud.github.io`. **Gotcha for the future:** a wildcard `*.plaidcloud.com → 35.192.209.39` (GCP) was catching `status` and routing it into non-GitHub infra; the explicit CNAME overrides the wildcard. If `status` ever 404s again, check that this explicit record still exists.
☐ **Enable "Enforce HTTPS"** in [Settings → Pages](https://github.com/PlaidCloud/plaidcloud-status/settings/pages) — the cert is valid now, so this is safe to turn on.
☐ Optional: Slack notifications (add the `NOTIFICATION_SLACK*` secrets, step 2 below); org-level domain verification to prevent takeover.

## Setup steps (completed — kept for reference)

1. **Create `GH_PAT` secret** (Settings → Secrets and variables → Actions). Upptime needs a Personal
   Access Token to commit results, open/close issues, dispatch workflows, and deploy Pages — the
   default `GITHUB_TOKEN` can't trigger other workflows and lacks the needed scope.
   - Classic PAT scopes: `repo` + `workflow`. Prefer a dedicated bot account over a personal token.

2. **(Optional) Slack alerts** — add these **repository secrets** (notifications are configured via
   secrets, not in `.upptimerc.yml`):
   - `NOTIFICATION_SLACK` = `true`
   - `NOTIFICATION_SLACK_WEBHOOK` = `true`
   - `NOTIFICATION_SLACK_WEBHOOK_URL` = `<your Slack incoming webhook URL>`
   - Email instead/also: `NOTIFICATION_EMAIL`=`true`, `NOTIFICATION_EMAIL_FROM`, `NOTIFICATION_EMAIL_TO`,
     plus provider creds (SES/Sendgrid/Mailgun/SMTP). Docs: https://upptime.js.org/docs/notifications

3. **Enable GitHub Pages.** After the first successful run (which creates the `gh-pages` branch),
   set Settings → Pages → Deploy from branch → `gh-pages` / root. Then set the custom domain to
   `status.plaidcloud.com` (matches `status-website.cname`).

4. **DNS.** Add `CNAME status.plaidcloud.com → plaidcloud.github.io`. Confirm the DNS provider is
   **independent of PlaidCloud infrastructure** (it should be — don't host DNS on the platform this
   page reports on). Enable "Enforce HTTPS" once the cert provisions.

5. **Go live.** Push to `main`. The **Setup CI** workflow runs on any push that touches
   `.upptimerc.yml`: it regenerates workflows, runs the first checks, builds the site, and deploys
   Pages. Watch the Actions tab.

6. **Verify.** Confirm the page renders all three components, then run a test incident (temporarily
   point one site at a URL that 404s, or open a manual incident issue) and confirm: issue auto-opens,
   Slack fires (if configured), the page updates, and recovery auto-closes.

## How the engine works (workflows)

All under `.github/workflows/`, calling `upptime/uptime-monitor`. They use `secrets.GH_PAT` (falling
back to `github.token`):

| Workflow | Trigger | Job |
| --- | --- | --- |
| `uptime.yml` | every 5 min | probe each site, commit up/down history, open/close incident issues |
| `response-time.yml` | daily | record response times |
| `graphs.yml` | daily | render response-time graphs |
| `summary.yml` | daily | regenerate the README status table |
| `site.yml` | daily + on change | build the Svelte site, deploy to `gh-pages` |
| `setup.yml` | on push to `.upptimerc.yml` | full bootstrap (template + response-time + readme + graphs + site + deploy) |
| `update-template.yml` | daily | sync the Upptime engine from upstream |
| `updates.yml` | daily | dependency updates |

## Gotchas — read before editing

- **`.templaterc.json` is narrowed to `.github/workflows/**`** (upstream default is all of `.github/**`).
  This keeps the engine auto-synced while protecting our `.github/ISSUE_TEMPLATE/` files from being
  overwritten by the daily template sync. **Don't widen it back** or the custom incident/maintenance
  templates will be clobbered.
- **README status markers:** Upptime rewrites only the `<!--live status-->` line and the content
  between `<!--start: status pages-->` / `<!--end: status pages-->`. Keep those markers; put any human
  content outside them. Everything else in `README.md` is preserved (it's not under `.github/`).
- **Don't edit workflow files directly** — they're regenerated from `.upptimerc.yml` by the template
  sync. Change behavior via `.upptimerc.yml`.
- **`slug` values are stable identifiers** used for issue labels and `expectedDown:`. Renaming a slug
  orphans existing incident references — avoid it.
- **Public repo:** never put secrets, customer data, or internal infra detail in issues or config.

## Known limitations of this approach

- **~5-minute check cadence** (GitHub Actions cron minimum) — not sub-minute; brief blips may be missed.
- **No native end-user email-subscription management** — subscribers use the GitHub issues Atom feed
  (`/issues.atom`) or Slack. If managed email subscriptions become a hard requirement, that's the one
  place a hosted SaaS (Instatus / Better Stack) would beat this; revisit then.
- **Single vantage point** (GitHub runner region) — a GitHub↔site network blip can cause a false
  positive. Tune check strictness in `.upptimerc.yml`; keep internal alerting as a secondary signal.

## Reference

- Config keys: https://upptime.js.org/docs/configuration
- Notifications: https://upptime.js.org/docs/notifications
- Upptime source: https://github.com/upptime/upptime
