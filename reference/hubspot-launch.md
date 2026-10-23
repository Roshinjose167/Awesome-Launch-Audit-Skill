# Launch Audit — HubSpot CMS Notes

Extra behavior when the audited site is HubSpot CMS (staging hosts like `explore.<client>.com`, `*.hs-sites.com`, `*.hubspotpagebuilder.com`; telltales: `hs-scripts.com`, `hsforms`, `hubspotusercontent`, `/_hcms/` paths).

## Domain-flip mechanics (IDX area)

- HubSpot rewrites canonicals + sitemap `<loc>` automatically when the page's primary domain changes. Canonicals on the temp domain are therefore **expected** — file the Blocker as "verify on prod at cutover", never as "fix canonicals now".
- robots.txt on HubSpot ships with preview-path disallows (`/hs/`, `/_hcms/preview`…) — normal, don't flag. The missing `Sitemap:` line is still a real finding.
- The 404/system pages (404, password-prompt, search) are configured per-domain under **Settings → Website → Pages → System pages** — they do NOT automatically inherit the new theme. Cloned portals commonly still show the source template's branding here (e.g. a site cloned from another client's template showing that client's logo). Check the 404 page explicitly (CMS area, Medium).

## Forms (FRM area)

- HubSpot forms post to `api.hsforms.com/submissions/…` or `…/submit/formsnext/multipart/<portalId>/<formGuid>`. Extract the **portal ID and form GUID** into the evidence — it lets the client find the form in their portal instantly.
- Form submissions create CRM contacts — this is why the audit **never test-submits**. The human-confirm item covers: inbox/CRM routing, internal notification, autoresponder.
- Spam protection: HubSpot's built-in CAPTCHA is off by default per-form — its absence is a real Medium finding.

## Code alerts (CMS area)

- Page source referencing `app.hubspot.com/code-alerts/<portalId>` means the portal has (or had) unresolved template/HubL errors. File High; the human confirms in **Design Manager / code alerts** — you cannot see the alert contents from outside.

## Security headers (SEC area)

- HubSpot sets `Content-Security-Policy: upgrade-insecure-requests` by default and offers **limited** header customization (Settings → Website → Pages → security, and reverse-proxy setups). Always append the platform caveat: "(HubSpot default; hardening is limited on the platform — confirm what's achievable)". Don't prescribe arbitrary nginx-style header fixes.

## Staging lockdown (SEC area)

- HubSpot options the client actually has: password-protect pages, set the temp domain to `noindex` via Settings → Domains, or leave open. Frame the human-confirm item around those three.

## CRM hygiene (CMS area)

- Test form submissions and dummy contacts live in the portal CRM — purging is a human task in the portal, not something the audit does. Ask whether QA contacts should be purged before analytics/attribution go live.

## Trackers (ANL area)

- HubSpot's own analytics + `__hstc`/`hubspotutk` cookies load with the tracking code — they count as trackers for the consent-gating check. HubSpot has a built-in cookie banner (Settings → Privacy & Consent) that is **off by default** — if no banner renders, that's the likely reason; say so in the evidence.

## Team conventions

- Upload-validation and theme rules for a given HubSpot build live in that project's own theme skill — if the audit leads into *fixing* theme files, load that skill first; this skill only audits.
