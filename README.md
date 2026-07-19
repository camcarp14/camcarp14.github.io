# camcarp14.github.io — clarifysearch.com

The Clarify Search marketing site. One static page, no build step, served by
GitHub Pages at the apex domain in `CNAME`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire site — markup, CSS and JS in one file, no dependencies except Google Fonts |
| `CNAME` | `clarifysearch.com` — do not delete, GitHub Pages needs it to keep the custom domain |
| `robots.txt` | Crawl rules; explicitly allows answer-engine bots |
| `sitemap.xml` | Single-URL sitemap |

## Deploying

Push to `main`. Pages republishes in roughly 40–90 seconds. Verify with a
cache-buster, because CDN caching will otherwise show you the old page:

```sh
curl -sS -A "Mozilla/5.0 Chrome/131.0" "https://clarifysearch.com/?cb=$RANDOM" | grep "something you changed"
```

## Open items

(1 is resolved, kept for history.)

1. ~~HTTPS~~ **Done (2026-07-18).** A certificate is provisioned, Enforce HTTPS
   is on, and `http://` 301s to `https://`. All URLs here and in `index.html`,
   `robots.txt` and `sitemap.xml` point at `https://clarifysearch.com`.

2. **Lead notifications.** The form posts to the `submit-lead` Supabase Edge
   Function, which writes to `inbound_leads` in the clarify-outreach project.
   Nothing emails you when a lead lands — you have to open the CRM. Worth wiring
   a database webhook or an email send inside the function.

3. **`inbound_leads` row-level security.** The `anon` role can `SELECT` and
   `UPDATE` every row, and the CRM's anon key is public in its unprotected
   Netlify bundle. Migrate the CRM's reads to its signed-in session token, then
   restrict those policies to `authenticated`. Do not tighten the policies
   first — that breaks the CRM.

## Conventions

- The checkout is CRLF. Edits made elsewhere in LF should be converted on the
  way in (`sed 's/$/\r/'`) or the diff shows every line as changed.
- Motion is gated behind `prefers-reduced-motion`; keep new animation gated too.
- Pricing lives in the `PRICING` object in the inline script and is rendered per
  channel — edit it there, not in the markup, or the segmented toggle will
  overwrite you. The same applies to the free tier's `FREE` object.
- CTA links use `data-offer="<Tier> — <Channel>"` and preselect the contact
  form's service dropdown **by matching the option text**. A new CTA needs a
  matching `<option>` or the preselect silently does nothing.
