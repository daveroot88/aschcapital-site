# aschcapital.com

Source for the aschcapital.com site: the **Boutique RIA AUM Index** screener (index,
methodology, the ceiling, by size, by state, screener, disclosures), built from SEC Form
ADV Part 1 data.

- `index.html` — the single-page site
- `index.json` — the screener dataset
- `favicon.svg`, `og.png`, `logo.svg`, `robots.txt`, `sitemap.xml`, `_headers` — static assets
- `wrangler.jsonc` — Cloudflare Worker static-assets config
- `DEPLOY.md`, `LAUNCH-CHECKLIST.md` — deployment notes

The build pipeline (`ingest/` SEC fetch, the Cloudflare `worker/`, `deploy/` scripts) and
the labs apps that also appear on the live site (Prometheus, Attention, Build, AI
Operations, Recruiting, Trading DNA, IRA Calculator, Economics Plan, Memory Major) live
separately; each labs app is its own repo.
