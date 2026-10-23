<!-- Copy to <project>/AGENTS.md (agents.md standard) — works for OpenCode, Cline, Aider, and any agents.md-compatible agent. Self-contained. -->

# Launch Audit — Pre-Launch Website QA

When given a list of staging URLs to test, a "launch audit" request, or a pre-go-live QA task, run this audit and produce one deliverable: the **Launch Audit — Issue List** for the production domain, plus a **Human-confirm checklist**.

## Scope intake

Only item 2 may block the run with a question — everything else has a default:

1. **Page list** = seed only; discovery widens it.
2. **Staging host vs production domain** (e.g. `explore.client.com` → `client.com`). Title the report with the **production** domain. Ask if the prod domain is unknown.
3. **Platform** — infer from hostname provisionally (`explore.*`, `*.hs-sites.com` → HubSpot), confirm from telltales (`hs-scripts.com`, `hsforms`, `/_hcms/`) on the first fetched page.
4. **Replacing an existing site?** — default: probe `https://<prod>/` + its sitemap once; a live site there = the old site being replaced (RDR runs on its URLs; timeout/parked = that IS the RDR evidence). The question also goes on the human-confirm checklist.
5. **Markets** — default when unknown: US/CCPA framing, note "confirm any EU obligations" in ANL evidence.

## Discovery

- Fetch `robots.txt` + `sitemap.xml` on staging. `sitemapindex` → fetch every child sitemap and union URLs; sitemap 404 → IDX row (High) + fall back to crawling nav/footer links from seed pages. Audit **every** discovered page, not just the seed list.
- **Cap policy:** >~25 URLs → full catalog on the seed list + one page per template/URL pattern; lightweight checks (title, meta, canonical, status) on the rest; state the split in the report.
- Probe: `/llms.txt`, `/favicon.ico`, a guaranteed-404 URL (inspect the 404 template), legal slugs (`/privacy`, `/privacy-policy`, `/legal/privacy`, `/terms`, `/legal/terms`), dev slugs (`/test`, module-library pages), and the production domain (see intake item 4).

## Rules of engagement

- **Read-only.** Never submit forms, create CRM records, or publish/unpublish. State changes become human-confirm items.
- **Evidence or it didn't happen** — char counts, contrast ratios with hexes, KB, request counts, exact URLs, header values.
- **Unverified ≠ passed.** A check that couldn't run still appears, severity `—` or as a spot-check — never silently dropped.
- **Staging ≠ broken.** Canonicals/sitemap on the staging host = Blocker worded "must auto-flip at domain publish; verify on prod at cutover", not a defect.
- **Kill false positives** — exclude text-over-photo contrast flags (and say so); probe alternate slugs before declaring a page missing; verify `#anchor` links target real elements before calling them dead.
- **Count what you cap** — if you sample, say what was skipped.

## Check areas (ID prefixes)

- **SEO** — titles (duplicates across pages = High; generic working titles = Medium; report char counts); meta descriptions (missing / >160ch / duplicated = Medium, exact counts); JSON-LD (no sitewide Organization/BreadcrumbList = Medium; note which pages carry which types); headings (one H1, no level skips, no empty/duplicate headings = Medium); image alts (count `alt=""`, note "confirm decorative vs meaningful" = Medium); dead links (`href="#"`, empty href on CTAs = Medium — but in-page modal triggers like `#trigger-contact-form` are fine if the target exists, and even with a missing target id HubSpot module JS often binds by href selector, so file those as unverified → click-test, not dead); favicon not linked = Low; missing og:image / weak twitter:card = Low (record the card value — `twitter:card=summary` with no image is still weak previews).
- **IDX** — canonicals (staging host = Blocker, cutover-verify wording); sitemap hygiene (dev/test pages listed = Medium; missing pages; 404ing entries); robots.txt missing `Sitemap:` line = Medium (platform preview disallows are normal).
- **PERF** — page weight / request count / image count per page (homepage >1.5 MB or >150 req = row; interior pages +25% slack); oversized images (served vs displayed px; count them — without a browser, compare intrinsic dimensions via `sips`/`identify` against `<img>` width/height/srcset, and say when displayed-size comparison was skipped); ≈1 MB single assets by name; missing width/height = Medium, "compress/resize + set dimensions". Core Web Vitals: report only what you measured (CLS, load, TTFB); if LCP/FCP injection fails, severity `—` + "re-measure with PageSpeed/field data on prod".
- **SEC** — CSP with no source directives = High; staging public + crawlable + no noindex = High (human decides lockdown); no X-Frame-Options/frame-ancestors = Medium; nosniff/Permissions-Policy/HSTS = Low combined row. On HubSpot append "(hardening is limited on the platform — confirm what's achievable)".
- **A11Y** — WCAG AA contrast on **solid backgrounds only**, 4.5:1 normal / 3:1 large, each failing pair with hexes + ratio = Medium (without a browser: derive pairs from theme CSS only where unambiguous, else unverified → Spot-checks); keyboard/focus = Medium "not verified via automation" → Spot-checks.
- **FRM** — locate lead forms, verify render + client validation, **never submit**. Without a browser, HubSpot forms are JS-embedded — extract portal ID + form GUID from the hsforms embed and fold render/validation/required-fields into the Blocker wording as unverified. Blocker worded: "Submit one test and confirm it lands in the right inbox/CRM + internal notification + autoresponder. Not auto-submitted." No reCAPTCHA/honeypot = Medium. Only-email-required = Low "confirm intended capture".
- **FUNC** — crawl every `<a href>` on every page, status-check internal targets; external URLs HEAD once each (follow redirects, ~5s timeout, 4xx/5xx = Low, connection failure = unverified). Footer 404s sitewide = Blocker (list label → path each); mis-mapped nav (About → /contact which 404s) = High; nav placeholders pointing at `/` = High "unfinished nav wiring"; body cards/tiles 404 = High. Mobile: try 375/768; if the environment can't shrink, say exactly what blocked it = Medium → Spot-checks.
- **CNT** — Privacy Policy missing at footer target AND all common slugs = **Blocker**; Terms = High; duplicate/hidden draft sections (repeated H2 blocks, hidden placeholder blocks) = Medium; leftover dev pages live = Medium (cross-ref IDX/CMS); lorem/placeholder copy by visibility.
- **AEO** — `/llms.txt` 404 = Low, optional.
- **RDR** — old domain = wherever the replaced site currently lives, usually the production domain itself pre-cutover. Old sitemap → old→new 301 map. If old domain unreachable, that IS the evidence = High: "provide old URL inventory (old sitemap / GSC top pages)".
- **ANL** — trackers firing with no consent banner/CMP = High "confirm CCPA/GDPR obligations"; tag inventory with IDs (GA4 `G-…`, GTM, HubSpot, Meta/LinkedIn/Clarity) = Medium, verified on prod after cutover; call out expected-but-absent tags.
- **CMS** — code-alert references in source = High (human checks Design Manager); unpublish test pages = Medium; purge QA/test CRM data = Medium; brand settings (favicon, 404/system pages showing legacy/cloned branding) = Medium.
- **EML** — SPF/DKIM/DMARC/MX owned by the ESP team = Low, "confirm mail still flows after cutover".

## HubSpot specifics

- Canonicals + sitemap `<loc>` auto-flip when the primary domain changes — audit verifies at cutover.
- Forms post to `api.hsforms.com` / `…/formsnext/multipart/<portalId>/<formGuid>` — put portal ID + form GUID in the evidence. Built-in CAPTCHA is off by default (real finding). Submissions create CRM contacts — another reason never to test-submit.
- `app.hubspot.com/code-alerts/<portalId>` in source = unresolved template/HubL errors (High, human-confirm).
- 404/system pages are configured per-domain and keep cloned-template branding — check the 404 page explicitly.
- HubSpot's cookie banner (Settings → Privacy & Consent) is off by default — likely reason no CMP renders.
- Staging lockdown options: password-protect, noindex the temp domain, or leave open by intent.

## Output format

Title: `Launch Audit — Issue List: <production-domain>`

Table columns exactly: `ID | Area | Check | Severity | Evidence / note`.
- IDs `<AREA>-<NN>`, numbered in discovery order, stable across re-audits.
- Severity: **Blocker** (launch must not happen until closed/confirmed) → **High** (fix before launch) → **Medium** (before or right after) → **Low** → **—** (informational/unmeasured).
- Sort: **Area, then Severity (Blocker→High→Medium→Low→—), then ID**; resolved rows below open rows within each area. State the sort rule under the table.
- Return the report inline in the response; write a file only if asked.
- Check column: short imperative failure name. Evidence column: URLs, numbers, exclusions, cutover framing, fix hint, cross-references ("pairs with IDX-01 / CMS-02").

Then the **Human-confirm checklist** in four groups — **Routing & data / Build & pages / Compliance & environment / Spot-checks** — each item leading with its issue ID(s) and asking a direct question (e.g. "FRM-01 — Test-submit the homepage form: did it reach the right inbox/CRM, and did the internal notification + autoresponder both fire?").

On re-audit: ask for (or locate) the prior Issue List first — if unavailable, say so and number fresh from 01. Otherwise keep IDs, mark fixed rows Resolved with verification evidence, sort them below open rows, and lead with a diff summary (N resolved / N open / N new).
