# Deploying aschcapital.com

**Where things stand.** The domain is registered at GoDaddy and currently serves a
**GoDaddy Websites + Marketing (Website Builder 8.0)** site — the default template with
placeholder copy still in it. That product has no file manager, no FTP, and no way to
upload custom HTML or a 1.7 MB JSON file, so the index cannot live there. The plan is to
keep the domain at GoDaddy and move DNS to Cloudflare, which hosts the site free.

**Your DNS today** (snapshot in `deploy/dns-before.txt`, keep it for rollback):

| Record | Value |
|---|---|
| Nameservers | `ns53.domaincontrol.com`, `ns54.domaincontrol.com` (GoDaddy) |
| A (apex) | `13.248.243.5`, `76.223.105.230` (GoDaddy Website Builder) |
| www | CNAME → `aschcapital.com` |
| MX | **none — no email is configured on this domain** |
| TXT | none |

No MX and no TXT means this is a clean cutover with **no email to break**. That is the
usual way a nameserver change goes wrong, and it does not apply here.

---

## Step 1 — Build the bundle

```bash
cd output/aschcapital
./ingest/refresh.sh          # optional, only if you want fresher SEC data
python3 ingest/build_site.py # writes dist/
```

`dist/` is the entire website, 1.86 MB:

```
index.html              100 KB   (30 KB gzipped)
data/index.json        1.70 MB   (586 KB gzipped, ~466 KB brotli)
og.png                   59 KB   social card
favicon.svg / apple-touch-icon.png
robots.txt / sitemap.xml
_headers                         Cloudflare + Netlify config
.htaccess                        Apache config, unused on Cloudflare
```

Preview exactly what will ship before uploading:

```bash
cd dist && python3 -m http.server 8124
./deploy/verify.sh http://localhost:8124
```

## Step 2 — Create the Cloudflare account and upload

You need to do the account and login steps yourself; I can't create accounts or enter
credentials.

1. Sign up at **dash.cloudflare.com** (free plan is all this needs).
2. **Workers & Pages → Create → Pages → Upload assets.**
3. Project name: `aschcapital`. Drag the whole **`dist` folder** in. Deploy.
4. You get a URL like `aschcapital.pages.dev`. **Test it now, before touching DNS:**

```bash
./deploy/verify.sh https://aschcapital.pages.dev
```

Everything should pass, and `index.json` should now report `content-encoding: br`.
If this URL works, the site works. DNS is the only thing left.

## Step 3 — Move the domain's DNS to Cloudflare

An apex domain (`aschcapital.com`, no `www`) on Cloudflare Pages requires the DNS zone to
be on Cloudflare, because apex records can't be CNAMEs and Cloudflare uses CNAME
flattening to make it work. So the nameservers have to change.

1. Cloudflare dash → **Add a site** → `aschcapital.com` → **Free** plan.
2. Cloudflare scans your existing records and shows what it found. It will import the two
   GoDaddy A records. **Leave them for now**, Step 4 replaces them.
3. Cloudflare gives you two nameservers, something like
   `xxx.ns.cloudflare.com` / `yyy.ns.cloudflare.com`. Copy both.
4. In **GoDaddy → My Products → Domains → aschcapital.com → Manage DNS → Nameservers →
   Change → I'll use my own nameservers**. Paste the two Cloudflare ones. Save.

Propagation is usually minutes; GoDaddy warns it can take up to 48 hours. Watch it:

```bash
watch -n 30 'dig +short NS aschcapital.com'
```

When that returns the Cloudflare nameservers, the zone is live on Cloudflare.

## Step 4 — Attach the domain to the Pages project

1. **Workers & Pages → aschcapital → Custom domains → Set up a custom domain.**
2. Add `aschcapital.com`. Then add `www.aschcapital.com` as well.
3. **This is the one gotcha:** delete any leftover **A records for the apex** in
   Cloudflare's DNS tab (the `13.248.243.5` and `76.223.105.230` imported in Step 3).
   Adding the custom domain creates the correct record automatically, and a stale A record
   pointing at GoDaddy will win and keep serving the placeholder.
4. **SSL/TLS → Overview → Full (strict).** The certificate issues automatically and
   usually takes a few minutes.

## Step 5 — Verify and clean up

```bash
./deploy/verify.sh https://aschcapital.com
./deploy/verify.sh https://www.aschcapital.com
```

Then, in GoDaddy, **cancel the Websites + Marketing subscription** so you stop paying for
the placeholder. Cancelling that product does **not** cancel your domain registration —
they are separate line items. Leave the domain registration alone.

**Rollback**, if you ever want the placeholder back: at GoDaddy set the nameservers back
to `ns53.domaincontrol.com` / `ns54.domaincontrol.com`. Everything in
`deploy/dns-before.txt` returns.

---

## Monthly refresh

The SEC posts a new Form ADV full file in the first week of each month.

```bash
./ingest/refresh.sh            # fetch + rebuild data/index.json  (~2 min)
python3 ingest/build_site.py   # rebuild dist/
./deploy/deploy.sh             # push to Cloudflare Pages
./deploy/verify.sh https://aschcapital.com
```

`deploy/deploy.sh` uses `wrangler` and needs a Cloudflare API token, which you create at
**dash.cloudflare.com/profile/api-tokens** (template: *Edit Cloudflare Workers*, or a
custom token with Account → Cloudflare Pages → Edit). Put it in your shell profile:

```bash
export CLOUDFLARE_API_TOKEN=...
export CLOUDFLARE_ACCOUNT_ID=...
```

Never commit that token. Until you set it up, re-uploading the `dist` folder through the
dashboard does the same job.

Cloudflare Pages keeps every deployment, so a bad data refresh is a one-click rollback in
the dashboard.

---

## Before you point DNS — three things to settle

**1. The corrections mailbox does not exist yet.** The Disclosures screen and the footer
now publish `corrections@aschcapital.com`, and your domain has **no MX records**, so that
address will bounce. Either add email to the domain (Cloudflare Email Routing forwards to
an existing inbox, free, and is the easiest fix) or change `CONTACT` at the top of
`ingest/build_site.py` to an address that already works, then rebuild. Publishing a
correction channel that silently bounces is worse than not offering one.

**2. Named advisers plus derived numbers will draw objections.** The filed figures are
unimpeachable, but "organic growth" is our arithmetic attributed to a named firm, and some
of the objections will be legitimate: an acquisition or a registration restructuring shows
up as growth and Form ADV can't distinguish them. That is why the correction channel
matters and why it should exist on day one, not after the first complaint.

**3. You are not a registered adviser, and that helps.** I checked the full SEC register:
zero matches for "Asch" among the 17,018 SEC-registered advisers. This publishes as
research, not as a registered adviser making comparative claims about competitors, which
would bring the SEC Marketing Rule into scope. If Asch Capital ever registers, revisit
this page with counsel first.

## Not recommended, for the record

- **GoDaddy cPanel hosting** works (the `.htaccess` in `dist/` is written for it) but costs
  roughly $100–150/yr, has no CDN, no brotli, and no deployment rollbacks.
- **Embedding in Website Builder** via a custom-code section cannot work: no root-level
  files, no way to serve `data/index.json`, and no routing.
