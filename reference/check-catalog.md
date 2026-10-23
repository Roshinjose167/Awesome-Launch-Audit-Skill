# Launch Audit — Check Catalog

Run every check below on every audited page unless marked *sitewide* (run once) or *conditional*. Number issues `<AREA>-<NN>` in the order found. Severity guidance is a default — evidence can move a finding up or down one step (say why in the note).

---

## SEO — Search optimization

### Titles
- Extract `<title>` from every page. Fail on:
  - **Duplicate titles across pages** (e.g. a service page reusing another template's title) — High. This usually means a wrong/leftover template title.
  - **Generic working titles** used verbatim ("Industry Overview Page", "Projects Overview Page") — Medium.
  - Missing brand suffix, <30 or >60 chars — fold into the above rows as evidence (report char counts).
- Evidence format: `/services/automation renders title "Industry Overview Page" (22ch, no brand) — same title as /industry-overview-page.`

### Meta descriptions
- **Missing/empty** — Medium. List every affected URL.
- **Too long** (>160 chars — will truncate) or too short (<70) — Medium. Report exact char count.
- Duplicated across pages — Medium.

### Structured data (JSON-LD)
- Parse every `<script type="application/ld+json">`; note which pages carry which types.
- **No sitewide `Organization`** schema — Medium.
- **No `BreadcrumbList`** on interior pages — fold into the same row.
- Page-type schema present but broken JSON / wrong page (e.g. FAQPage on a page with no FAQ) — Medium.
- Evidence: name pages with zero JSON-LD and pages that do carry types.

### Heading structure
- One `<h1>` per page (0 or 2+ = fail). No level skips (H1→H3, H2→H4). No empty headings. No duplicate section H2s (often a symptom of duplicated draft sections — cross-reference CNT).
- Medium; per-page evidence with counts.

### Image alts
- Count `alt=""` and missing-alt images per page. Client-logo strips are the usual bulk.
- Medium, with total count and a "confirm decorative vs meaningful" note — empty alt is *correct* for decorative images, so this is a confirm item, not an automatic fix.

### Placeholder / dead links
- `href="#"`, `href=""`, `javascript:void(0)` on visible CTAs ("Learn More", "View Case Study").
- Medium. Count per page; name the worst offenders.
- Exception: `#anchor` links that actually target an in-page element (e.g. `#trigger-contact-form` opening a modal) are fine — verify the target exists before flagging.
- Fragment CTAs whose target id does NOT exist are still not automatically dead on HubSpot: module JS often binds by href selector (`[href="#trigger-…"]`), so a renamed section id changes nothing. File as unverified → click-test spot-check, not as a dead link.

### Favicon
- `<link rel="icon">` present on published pages? A favicon existing in the file manager but not linked is still a fail. Low.

### Social tags
- `og:title`, `og:description`, `og:image`, `twitter:card`. Missing `og:image` sitewide = poor link previews. Record the card *value* — `twitter:card=summary` with no image is still weak previews. Low.

---

## IDX — Indexing

### Canonicals *(sitewide pattern + per-page)*
- Every page should self-canonical. On a staging host, canonicals pointing at the staging domain are **expected** — file as **Blocker** worded "must auto-flip to <prod-domain> at domain publish; verify on prod at cutover", not as a defect.

### Sitemap hygiene *(sitewide)*
- Fetch `sitemap.xml`. If it's a `sitemapindex`, fetch every child sitemap and union the URLs before checking. If it 404s entirely — High (staging sitemap missing; discovery falls back to crawling nav/footer links from the seed pages).
- Fail on: dev/test pages listed (`/test`, module-library pages) that return 200 publicly — Medium (pairs with CNT/CMS unpublish items); seed-list pages missing from the sitemap; sitemap URLs that 404.

### robots.txt *(sitewide)*
- Must contain a `Sitemap:` line (add prod sitemap URL at cutover) — Medium.
- Check for accidental `Disallow: /` or blocks on real content; HubSpot preview-path disallows are normal.

---

## PERF — Performance

### Page weight & images
- Per page: total transfer size, request count, image request count. Homepage >1.5 MB or >150 requests deserves a row; interior pages get the same thresholds with ~25% slack.
- Oversized images: served dimensions vs displayed dimensions (e.g. 720px logo displayed at 124px); single assets ≈1 MB by name. Count how many images are oversized.
- Without a browser: download image assets, read intrinsic dimensions (`sips -g pixelWidth pixelHeight` on macOS / ImageMagick `identify`), compare against the `<img>` tag's `width`/`height`/`srcset` hints. If displayed size is unknowable, report intrinsic size + file KB only and say the displayed-size comparison was skipped.
- Missing `width`/`height` attributes (CLS risk) — fold into evidence.
- Medium. Fix note: "compress/resize + set dimensions".

### Core Web Vitals
- With browser tools: capture LCP, FCP, CLS, TTFB, load time. Post-load injection often can't capture LCP/FCP — if so, report what you *did* measure (CLS, load, TTFB) and file the row at severity `—` with "re-measure with PageSpeed/field data on prod".
- Never report vitals you didn't measure.

---

## SEC — Security

*Platform caveat: on HubSpot, header hardening is limited — always append "(confirm what's achievable on the platform)" rather than demanding arbitrary headers.*

### CSP *(sitewide, from response headers)*
- `Content-Security-Policy` with no source directives (`default-src`/`script-src`) = no XSS mitigation — High.

### Staging exposure *(sitewide)*
- Staging host loads with no auth AND robots allows crawl AND no `noindex` → risk of temp domain being indexed and competing with prod — High. Human-confirm: "lock down (noindex/basic-auth) or open by intent?"

### Clickjacking
- No `X-Frame-Options` and no `frame-ancestors` in CSP — Medium.

### Minor headers
- `X-Content-Type-Options: nosniff`, `Permissions-Policy`, HSTS — Low, one combined row.

---

## A11Y — Accessibility

### WCAG AA contrast
- Compute ratios for text on **solid backgrounds only** (body text, buttons, stat labels). Threshold 4.5:1 normal text, 3:1 large text.
- Report each failing pair with hexes and ratio: `white on orange #FA582D = 3.23:1 (body + "Learn More")`.
- **Exclude text over photos/gradients as false positives — and say you excluded them.**
- Without a browser: derive pairs from theme CSS (button background + text color variables) only where unambiguous; otherwise mark contrast unverified and move it to Spot-checks alongside keyboard/focus.
- Medium.

### Keyboard & focus
- Rarely automatable — file as Medium with "not verified via automation" and add to the Spot-checks section of the human-confirm checklist: tab order + visible focus rings on nav, form, CTAs.

---

## FRM — Forms

### Lead-form routing — **never auto-submit**
- Locate every lead form (HubSpot forms: `…/submit/formsnext/…` endpoints). Verify it renders and client-side validation fires (e.g. email required). **Do not submit.**
- Without a browser: HubSpot forms are JS-embedded (`hsforms` script) so raw HTML has no fields — detect the embed script, extract portal ID + form GUID (see `hubspot-launch.md`); render/validation/required-fields become unverified and fold into the FRM-01 Blocker wording rather than separate unverifiable rows.
- File as **Blocker** worded as the human task: "Submit one test and confirm it lands in the right inbox/CRM + internal notification + autoresponder. Not auto-submitted."

### Spam protection
- No reCAPTCHA/hCaptcha/honeypot detected — Medium ("add before launch to avoid bot spam").

### Required fields
- List which fields are required vs optional; if only email is required, file Low: "confirm this matches intended capture".

---

## FUNC — Functionality

### Full link crawl
- Crawl every `<a href>` on every audited page (nav, footer, body cards). HEAD/GET each internal target; record status.
- External links: HEAD each unique external URL once (follow redirects, ~5s timeout); 4xx/5xx → Low row; connection failures → note as unverified, don't retry.
- **Footer links that 404 sitewide** — Blocker (every visitor sees them). List each: label → path → status.
- **Nav items mis-mapped** (e.g. "About" → `/contact`, which itself 404s) — High.
- **Nav items pointing at `/` as placeholders** (dropdown children, "Contact Us") — High, "unfinished nav wiring". Exclude in-page modal triggers (`#trigger-…`) — verify they work and say they're fine.
- **Body cards / tiles that 404** (industry tiles, case-study cards) — High.

### Mobile/tablet rendering
- Try 375/768 viewports with browser tools. If the environment won't render below some width, say so exactly ("wouldn't render below ~1265px") and file Medium: viewport meta present? mobile-nav toggle present? desktop horizontal overflow? Then hand to Spot-checks. For a deep pass, chain the `mobile-responsiveness` skill.

---

## CNT — Content

### Legal pages
- Privacy Policy: footer link target 404s AND not found at any common slug (`/privacy`, `/privacy-policy`, `/legal/privacy`) → **Blocker** ("required legal page missing before public launch").
- Terms of Service: same probe → High.

### Duplicate / hidden draft content
- Duplicated section headings ("ACI Projects" ×2), hidden placeholder blocks ("Project Target One/Two/Three"), leftover draft markup — Medium.

### Leftover dev pages
- `/test`, module-library pages published and reachable — Medium. Pairs with IDX sitemap row and CMS unpublish item; cross-reference IDs.

### Placeholder copy
- Lorem ipsum, "Coming soon", template strings — severity by visibility.

---

## AEO — AI engines

- `/llms.txt` 404s — Low, "optional AI-crawler aid".

---

## RDR — Redirects *(conditional: replacing an existing site)*

- **Old domain** = wherever the site being replaced currently lives — usually the *production domain itself* pre-cutover (probed once in Phase 1), sometimes a separate legacy domain the user names.
- Fetch the old domain's sitemap / homepage. If unreachable (parked, timing out), report that as the evidence — High: "provide old URL inventory (old sitemap / GSC top pages) to build old→new 301s. Verified live post-cutover."
- If retrievable: build the old→new URL map skeleton and flag unmapped old URLs.

---

## ANL — Analytics

### Consent gating
- Detect trackers (GA4 `G-…`, GTM `GTM-…`, HubSpot analytics, Meta/LinkedIn/Clarity). If they fire with no consent banner/CMP detected — High: "confirm CCPA (and any EU) consent obligations before go-live".

### Tag inventory & prod verification
- List every detected tag with its ID. Medium row noting: pageview firing, conversion recording, and consent behavior get verified **on prod after cutover**; confirm the intended tag set (call out common tags that are *absent*).

---

## CMS — Platform housekeeping

See `hubspot-launch.md` for the HubSpot-specific versions. Generic:

- **Build/template errors** — any error console/code-alert reference in page source — High, human-confirm in the CMS UI.
- **Unpublish test pages** — Medium (pairs with IDX/CNT rows; cross-reference).
- **Purge QA/test CRM data** before go-live — Medium, human task.
- **Brand settings** — favicon set; 404/system pages still showing another brand's template branding — Medium.

---

## EML — Email

- SPF/DKIM/DMARC/MX are owned by the ESP/marketing-platform team — file one Low row: "confirm with them that mail still flows after cutover". Don't audit DNS yourself unless asked.
