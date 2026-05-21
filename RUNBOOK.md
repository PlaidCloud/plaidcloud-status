# PlaidCloud Status — Incident Runbook

How to communicate during a PlaidCloud incident or planned maintenance using this status page.
The page is public, so everything here is written for an external audience.

## Golden rules

1. **First update within 15 minutes** of confirming an incident. A short "we're aware and investigating" beats silence.
2. **Impact first, plain language.** Lead with what users experience, not internal cause. No jargon, no component code names.
3. **No secrets, customer data, or internal infrastructure detail** — this repository and page are public.
4. **Update regularly** — at least every 30–60 min during an active incident, even if the update is "still working on it."
5. **Always close the loop** — mark resolved and, for major incidents, follow up with a post-mortem.

## Severity levels

| Severity | Meaning | Example |
| --- | --- | --- |
| **Degraded performance** | Working but slow or partially impaired | Slow page loads, elevated latency |
| **Partial outage** | One or more components unavailable; others fine | Docs site down, app and website up |
| **Major outage** | Core platform unusable for many/all users | App or website down |
| **Maintenance** | Planned, announced work window | Planned site deployment |

## Component slugs

Use these as issue **labels** (to link an incident to a component) and in the maintenance `expectedDown:` list.

| Component | Slug |
| --- | --- |
| Web App | `app` |
| Documentation | `docs` |
| Website | `website` |

## Incidents

### Automatic incidents

When a monitored health check fails, Upptime **auto-opens an issue** and posts it on the status page; when the check recovers, the issue **auto-closes**. You don't need to do anything for the detection itself — but for anything user-visible you should still **add human updates as comments** so customers get context, not just a red box.

### Manual incidents

For issues a synthetic check can't see (e.g. a known data problem, a partner outage), open one yourself:

1. **New issue → "Incident" template.**
2. Set a clear, user-facing **title** (e.g. `[Incident] Logins failing for some users`).
3. **Add a label** for each affected component slug (e.g. `app`).
4. Fill in **Status**, **Severity**, **Affected**, **Started (UTC)**, **Impact**, and the first **update**.
5. Post each subsequent update as a **new comment**, advancing the status:
   **Investigating → Identified → Monitoring → Resolved**.
6. **Close the issue** to mark it resolved on the page.

### Update template (paste as a comment)

```
**Update — <HH:MM UTC>** (<Investigating|Identified|Monitoring|Resolved>)
<What changed since the last update, in plain language.>
Next update by <HH:MM UTC>.
```

## Scheduled maintenance

1. **New issue → "Scheduled Maintenance" template.**
2. Edit the HTML comment block with ISO-8601 **UTC** times and the affected slugs:
   ```
   start: 2026-05-25T02:00:00.000Z
   end: 2026-05-25T04:00:00.000Z
   expectedDown: docs
   ```
3. Describe **What / When / Impact** in user terms.
4. Open it **ahead of time** (ideally 24–72h) so subscribers see it coming.
5. During the window, the page shows it as active maintenance; **close the issue** when finished.

## Post-mortems (major outages)

Within a few business days of a major outage, post a follow-up comment (or linked write-up) covering:

- **What happened** and the user-facing impact + timeline.
- **Root cause** at a level appropriate for a public audience.
- **What we're doing** to prevent recurrence.

## When the status page itself misbehaves

- **False positive** (page red but service is fine): usually a single-vantage-point network blip between GitHub and PlaidCloud, or a too-strict check. Verify the endpoint yourself; tune `expectedStatusCodes` / `maxResponseTime` / body matchers in [.upptimerc.yml](./.upptimerc.yml).
- **Checks not running / page not updating:** check the Actions tab. Most failures trace back to the `GH_PAT` secret (expired/insufficient scope). See [CLAUDE.md](./CLAUDE.md).
- Keep PlaidCloud's internal alerting as a **secondary** signal that prompts a human to post here — never the page's only driver, since internal alerting can be down with the platform.
