---
name: launch-audit
description: Pre-launch website QA audit that produces the "Launch Audit — Issue List" (ID / Area / Check / Severity / Evidence table + human-confirm checklist). Sweeps SEO (titles, metas, schema, headings, alts, dead links, favicon, OG), indexing (canonicals, sitemap, robots), performance (page weight, oversized images, CWV), security headers, staging exposure, WCAG contrast, lead forms (render-only, never auto-submit), broken nav/footer links, legal pages, duplicate draft content, llms.txt, redirect-map prep, analytics/consent gating, CMS cleanup, and email-DNS handoff. TRIGGER on "launch audit", "test these pages", "pre-launch QA", "audit before go-live", "QA the staging site", "cutover checklist", or a pasted list of staging URLs to test before a domain launch.
version: 1.0.0
author: Roshin Jose
author_email: roshinjose1@gmail.com
author_phone: "+919645228131"
user-invocable: true
argument-hint: "[staging-urls-or-domain] [production-domain?]"
license: MIT
---

# Launch Audit

A repeatable pre-launch QA audit for websites about to cut over to a production domain — typically a HubSpot CMS site on a staging host (`explore.<client>.com`, `*.hs-sites.com`) launching to the real domain. Output is a single deliverable: the **Launch Audit — Issue List** for the production domain, plus a **Human-confirm checklist** of items only a human can close.

## When to invoke

- User pastes a list of staging URLs and says "test these pages", "audit these", "QA before launch".
- User asks for a "launch audit", "pre-launch checklist", "go-live QA", or "cutover audit".
- A site is moving from a temp/staging domain to a production domain.

Skip when the user wants a single-topic check ("check mobile" → `mobile-responsiveness`; "is this accessible" → run only A11Y area) — this skill is the full sweep.

## Phase 0 — Scope intake

Lock these down before auditing. Only item 2 may block the run with a question — every other item has a default; never stall the audit on items 3–5:

1. **Page list** — the URLs given. Treat them as the *seed* list, never the whole scope: discovery (Phase 1) will widen it.
2. **Staging host vs production domain** — e.g. staging `explore.suturetex.com` → prod `automation-control.com`. The report is titled with the **production** domain. If no prod domain is given, ask; canonicals/sitemap/redirect checks depend on it.
3. **Platform** — infer provisionally from the hostname (`explore.*`, `*.hs-sites.com` → HubSpot); confirm from telltales on the first page fetched in Phase 1 (see [reference/hubspot-launch.md](reference/hubspot-launch.md)); ask only if telltales are ambiguous. HubSpot gets the extra checks in that file.
4. **Replacing an existing site?** — default: resolve it yourself in Phase 1 by probing the production domain once. Whatever the answer, the question also appears on the human-confirm checklist (RDR area).
5. **Markets** — determines consent obligations for the ANL area. Default when unknown: US/CCPA framing, with "confirm any EU obligations" in the ANL evidence.

## Phase 1 — Discovery

Widen the seed list before checking anything:

- Fetch `robots.txt` and `sitemap.xml` on the staging host. If the sitemap is a `sitemapindex`, fetch every child sitemap and union the URLs. If it 404s, file an IDX row (High) and fall back to crawling nav/footer links from the seed pages. Diff the result against the seed list — audit **every** discovered page, not just the ones pasted.
- **Cap policy:** if discovery yields more than ~25 URLs, run the full per-page catalog on the seed list plus one page per template/URL pattern, and lightweight checks only (title, meta description, canonical, status code) on the rest. State the split in the report — count what you cap.
- Probe standard slugs: `/llms.txt`, `/favicon.ico`, a guaranteed-404 URL (to inspect the 404 template), and common legal slugs (`/privacy`, `/privacy-policy`, `/legal/privacy`, `/terms`, `/legal/terms`).
- Probe for leaked dev pages: `/test`, `/all-modules-listed-here`, anything in the sitemap with a working-title slug.
- Probe the **production domain** once (`https://<prod>/` + `/sitemap.xml`). A live site there is the "old site" being replaced → the RDR area runs on its URL inventory. If it times out or is parked, that itself is the RDR evidence — note it, don't retry forever.

## Phase 2 — Run the checks

Work through every area in [reference/check-catalog.md](reference/check-catalog.md). Area codes and scope:

| Code | Area | Covers |
|------|------|--------|
| SEO | Search optimization | titles, meta descriptions, JSON-LD schema, heading structure, image alts, placeholder/dead links, favicon, og:/twitter tags |
| IDX | Indexing | canonicals (staging→prod flip), sitemap hygiene, robots.txt Sitemap line |
| PERF | Performance | page weight, request/image counts, oversized images, LCP/CLS/TTFB where measurable |
| SEC | Security | CSP directives, clickjacking headers, nosniff/Permissions-Policy, staging public+indexable |
| A11Y | Accessibility | WCAG AA contrast (solid backgrounds only), keyboard/focus spot-check flag |
| FRM | Forms | lead-form render + validation (**never submit**), spam protection, required fields |
| FUNC | Functionality | full nav/footer/card link crawl (404s), placeholder nav wiring, mobile rendering |
| CNT | Content | privacy/terms exist, duplicate/hidden draft sections, leftover dev pages, placeholder copy |
| AEO | AI engines | llms.txt |
| RDR | Redirects | old-site URL inventory → 301 map prep |
| ANL | Analytics | tag inventory, consent gating, prod-verification notes |
| CMS | Platform | build/template errors, unpublish test pages, purge test CRM data, brand/system-page settings |
| EML | Email | SPF/DKIM/DMARC/MX handoff note |

**Tooling ladder** — degrade gracefully, and record *which rung you used* for anything you couldn't verify:

1. **Browser tools** (if available): computed contrast ratios, form behavior, performance timings, viewport shrink for mobile, tracker/network inspection.
2. **`curl` + parsing**: headers (`curl -sI`), HTML head/body parsing (title, metas, JSON-LD, headings, alts, links), status codes for the link crawl, page-weight estimation from resource lists.
3. **Can't check at all** → the check still appears, marked unverified, usually as a human-confirm item.

## Rules of engagement

These are what make the audit trustworthy — do not skip them:

- **Read-only.** Never submit forms, create CRM records, publish/unpublish pages, or change settings. Anything requiring a state change goes on the human-confirm checklist (e.g. FRM-01 test submission).
- **Evidence or it didn't happen.** Every issue row cites concrete evidence: char counts, contrast ratios, KB sizes, exact failing URLs, header values, element counts. "Meta description too long" is worthless; "202 chars (truncates ~160)" is actionable.
- **Unverified ≠ passed.** If a check couldn't run (LCP injection failed, viewport wouldn't shrink), report it with severity `—` or as a spot-check item — never silently drop it, never guess.
- **Staging ≠ broken.** Canonicals/sitemap pointing at the staging host is *expected* pre-cutover — file it as a Blocker to *verify at cutover*, not a bug to fix now. Distinguish "broken today" from "must flip at launch" in every evidence note.
- **Kill false positives before reporting.** Contrast on text over photos/gradients is excluded (note the exclusion). A "missing page" isn't missing until common alternate slugs were probed. A nav link to `/` is only a placeholder if the label says otherwise.
- **Count everything you cap.** If you sample (e.g. checked 4 of 96 images in depth), say what was sampled and what was skipped.

## Phase 3 — Report

Assemble exactly per [reference/report-format.md](reference/report-format.md) and return the report inline in the response (write a file only if the user asks):

1. Title: `Launch Audit — Issue List: <production-domain>`
2. Issue table — columns `ID | Area | Check | Severity | Evidence / note`; IDs are `<AREA>-<NN>`; sort by **Area, then Severity (Blocker→High→Medium→Low→—), then ID**. Resolved rows sort below open rows within each area.
3. **Human-confirm checklist** — grouped **Routing & data / Build & pages / Compliance & environment / Spot-checks**, each item referencing its issue ID(s).

Severity ladder: **Blocker** (launch must not happen until closed/confirmed) → **High** (fix before launch) → **Medium** (fix before or immediately after launch) → **Low** (nice-to-have) → **—** (informational/unmeasured).

## Platform notes

HubSpot sites: also run [reference/hubspot-launch.md](reference/hubspot-launch.md) — domain-flip behavior, code alerts, system-page branding, CRM test-data purge, and the platform's header-hardening limits (don't file CSP findings without the platform caveat).
