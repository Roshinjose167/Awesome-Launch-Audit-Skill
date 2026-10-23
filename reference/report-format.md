# Launch Audit — Report Format

One deliverable, two parts: the **Issue List** table and the **Human-confirm checklist**. Title the report with the **production** domain even when the audit ran on staging.

```
Launch Audit — Issue List: <production-domain>
```

## Part 1 — Issue table

Columns, exactly:

| ID | Area | Check | Severity | Evidence / note |
|----|------|-------|----------|-----------------|

### IDs
- `<AREA>-<NN>`: `SEO-01`, `IDX-03`, `FRM-01`… Numbered in discovery order within each area. IDs are stable — never renumber on re-audit; resolved rows keep their ID.

### Severity ladder
| Severity | Meaning |
|----------|---------|
| **Blocker** | Launch must not happen until this is closed or explicitly confirmed (missing privacy policy, unverified lead-form routing, sitewide 404 footer, canonical/sitemap flip verification) |
| **High** | Fix before launch (duplicate titles, 404 nav targets, no CSP directives, indexable staging, ungated trackers, template errors) |
| **Medium** | Fix before launch or immediately after (most SEO hygiene, missing headers, contrast, spam protection, dev pages) |
| **Low** | Nice-to-have (favicon, og:image, llms.txt, minor headers) |
| **—** | Informational / could not be measured (state what *was* measured and where to re-measure) |

### Sort order
**Area, then Severity (Blocker→High→Medium→Low→—), then ID.** Resolved rows sort below open rows within each area. State the sort rule under the table.

### Check column
Short imperative phrase naming the failure, not the test: "Duplicate/incorrect `<title>`", "Footer links 404 sitewide", "Trackers fire with no consent gating".

### Evidence / note column
The row lives or dies here. Include:
- Exact URLs/paths affected, or "sitewide (every page)".
- Numbers: char counts, contrast ratios with hexes, KB/MB, request counts, image counts, link counts per page.
- What was excluded and why ("white-on-hero-photo flags excluded as false positives").
- Staging-vs-cutover framing where relevant ("Expected on temp domain; must auto-flip to <prod> at domain publish. Verify on prod at cutover.").
- The fix or next action when it isn't obvious from the check name.
- Cross-references to paired rows ("pairs with IDX-01 / CMS-02").

### Example rows (from a real audit)

| ID | Area | Check | Severity | Evidence / note |
|----|------|-------|----------|-----------------|
| SEO-01 | SEO | Duplicate/incorrect `<title>` | High | /services/automation renders title "Industry Overview Page" (22ch, no brand) — same title as /industry-overview-page. Wrong template title left in place. |
| IDX-03 | IDX | Canonical + sitemap on staging host | Blocker | All canonicals + sitemap loc = explore.suturetex.com. Expected on HubSpot temp domain; must auto-flip to automation-control.com at domain publish. Verify on prod at cutover. |
| A11Y-01 | A11Y | WCAG AA contrast failures | Medium | White on orange #FA582D = 3.23:1 (body + "Learn More"); white on red #E74C3C = 3.82:1 ("Talk to an Expert"). All below 4.5. (White-on-hero-photo flags excluded as false positives.) |
| FRM-01 | FRM | Lead form routing to inbox/CRM | Blocker | Homepage HubSpot form renders + validates (email required). Submit one test and confirm it lands in the right inbox/CRM + internal notification + autoresponder. Not auto-submitted. |
| FUNC-01 | FUNC | Footer links 404 sitewide | Blocker | Footer links 8 pages that 404: About Us /company, Industries /industries, News /news, Contact /contact, Privacy /legal/privacy, Terms /legal/terms, Site Map /site-map. Sitewide (every page). |
| PERF-02 | PERF | LCP not measurable via injection | — | Post-load injection couldn't capture LCP/FCP. CLS = 0 (good), load ≈2.8 s, TTFB fast (edge-cached). Re-measure LCP with PageSpeed/field data on prod. |

## Part 2 — Human-confirm checklist

Everything the audit could not close itself — state changes, business decisions, and manual spot-checks. Four fixed groups; each item leads with its issue ID(s) and asks a direct question:

```
## Human-confirm checklist

### Routing & data
- FRM-01 — Test-submit the homepage form: did it reach the right inbox/CRM, and did the
  internal notification + autoresponder both fire?
- CMS-03 — QA/test submissions and dummy contacts purged from the CRM?

### Build & pages
- FUNC-01–04, CNT-01/02 — The N nav/footer links that 404: not built yet, or should they exist now?
- CMS-01 — Any unresolved template/code-alert errors in the CMS?
- CNT-03/04 — OK to remove duplicate/hidden draft blocks and unpublish the dev pages?

### Compliance & environment
- SEC-05 — Lock staging down (noindex/basic-auth) until cutover, or leave open by intent?
- ANL-02 — Cookie-consent banner required for your markets? Trackers fire ungated now.
- RDR-01 — Existing site being replaced? Share its old URL list/sitemap for the redirect map.
- EML-01 — Confirm with the ESP/platform team that mail still flows after cutover.

### Spot-checks
- A11Y-02 — Keyboard tab order + visible focus on nav/form/CTAs.
- FUNC-05 — Mobile/tablet rendering (state the viewport limitation that blocked automation).
```

Group mapping for the less obvious areas: EML and post-cutover ANL verification items go under **Compliance & environment**; anything the automation couldn't observe (contrast without a browser, keyboard, mobile viewports) goes under **Spot-checks**.

## Re-audit behavior

On a follow-up run against the same site:
- Ask for (or locate) the prior Issue List first. If it's unavailable, say so and number from 01 as a fresh audit — never guess prior IDs.
- Keep all existing IDs; append new findings with the next number in each area.
- Mark fixed rows **Resolved** (add a Resolved note with the verification evidence) and sort them below open rows within their area.
- Diff summary at top: N resolved, N still open, N new.
