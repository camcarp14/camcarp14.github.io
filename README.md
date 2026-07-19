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

2. ~~Lead notifications~~ **Done (2026-07-18).** `submit-lead` emails
   clarifypaidsearch@gmail.com via Resend on every lead, with `reply_to` set to
   the sender so you can reply directly. Verified delivering. The lead saves
   first and a mail failure is logged, never shown to the visitor as a failure.
   The older `inbound-lead` function is retired to a 410 stub — it accepted
   POSTs from any origin with no rate limit. Restore from the Supabase
   dashboard if ever needed.

3. **`inbound_leads` row-level security — last one open.** The `anon` role can
   still `SELECT` and `UPDATE` every row. The blocker is gone: the CRM now sends
   its session token (`src/lib/supabase.js` uses `sessionToken || anon`), so the
   policies can finally be narrowed. Before running it, confirm the deployed
   CRM at clarify-outreach.netlify.app is on a build that includes that helper,
   then:

   ```sql
   drop policy "anon can read leads"   on public.inbound_leads;
   drop policy "anon can update leads" on public.inbound_leads;
   create policy "authenticated read leads"   on public.inbound_leads
     for select to authenticated using (true);
   create policy "authenticated update leads" on public.inbound_leads
     for update to authenticated using (true);
   ```

   Keep the `anon can insert leads` policy — it is what lets the public form
   write. Verify afterwards by loading the CRM's Inbound view; if it goes empty,
   the deployed bundle predates the session-token helper, so roll the policies
   back and redeploy the CRM first.

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
