# PlaidCloud Status

Public status page and incident history for the PlaidCloud platform, powered by [Upptime](https://upptime.js.org). Uptime checks run on **GitHub Actions**, incidents are tracked as **GitHub Issues**, and the page is served from **GitHub Pages**.

This runs **entirely on GitHub, independent of PlaidCloud infrastructure** — so the status page stays up even when PlaidCloud is down. That isolation is the whole point of a status page.

🔗 **Live page:** https://status.plaidcloud.com

## [📈 Live status](https://status.plaidcloud.com): <!--live status--> **Loading…**

<!--start: status pages-->
<!--end: status pages-->

> The table above is generated automatically by Upptime. Don't edit between the
> `status pages` markers — your changes will be overwritten.

## How it works

- Every ~5 minutes, GitHub Actions probes each site **from outside PlaidCloud** (so it sees what users see).
- A failing check **auto-opens a GitHub Issue** (an incident) and **auto-closes** it on recovery.
- Maintainers can also open incidents and maintenance windows **manually** via the issue templates.
- The Svelte status page is rebuilt and deployed to GitHub Pages on every change.

Monitored components are defined in [.upptimerc.yml](./.upptimerc.yml).

## Managing incidents & maintenance

Full workflow — severity levels, the 15-minute first-update rule, component slugs, communication style — is in **[RUNBOOK.md](./RUNBOOK.md)**.

Quick version:

- **Incident** — open a new issue with the *Incident* template, add the affected component's slug as a label (`app`, `docs`, `website`), post updates as comments, and close the issue to resolve.
- **Scheduled maintenance** — open a new issue with the *Scheduled Maintenance* template and fill in the `start` / `end` / `expectedDown` block.

## Subscribe

- **RSS/Atom (incidents):** https://github.com/PlaidCloud/plaidcloud-status/issues.atom
- **Watch** this repository for issue notifications.
- **Slack** alerts are delivered via repository secrets (see [CLAUDE.md](./CLAUDE.md)).

## Setup & operations

First-time setup, required secrets, DNS, and GitHub Pages configuration are documented in **[CLAUDE.md](./CLAUDE.md)**.

## Credits & license

Built on [Upptime](https://github.com/upptime/upptime) by Anand Chowdhary (MIT — see [LICENSE](./LICENSE)). Monitoring data in `history/` is under the [Open Database License](https://opendatacommons.org/licenses/odbl/1-0/).
