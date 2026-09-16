# Four new sections — status

**LIVE.** https://aschcapital.com, deployed 2026-08-24.
Worker `white-darkness-ab1e`, version `046832e9-45fb-4452-b0ee-e521a82fd8da`.
Roll back with `npx wrangler rollback` from `output/aschcapital`.

Everything works, including file intake. One thing is left, and it needs you.

## LIVE

**https://aschcapital.com**, Worker `white-darkness-ab1e`, version
`26b213fc-8a42-424a-9e33-538e4beeb03c`, deployed 2026-08-24.
Roll back with `npx wrangler rollback` from `output/aschcapital`.

Everything works: four sections, four skins, the artwork, file intake, and now
checkout and real email on the domain.

### Venmo checkout is on

`@David-Root-1`. Verified live: the assessment button opens Venmo prefilled at $1,500;
the two retainers at $1,750 and $3,000 with "(first month)" in the note and a
"Rather be invoiced" button beside each, because Venmo cannot bill on a schedule.

**`PAYMENT.venmo.business` is still `false` in `app/offers.js`.** Nothing in the code
reads it. The underlying question is real: if `@David-Root-1` is a personal profile,
running retainer payments through it is against Venmo's user agreement and the penalty
is frozen funds. Convert it before the first retainer, then flip the flag.

### Email on the domain

`corrections@aschcapital.com` and `hello@aschcapital.com` both forward to
`dave@cravecookies.com` through Cloudflare Email Routing. The site shows and links real
addresses now, and `dave@cravecookies.com` no longer appears anywhere on the page.

A test draft is in your Gmail, "Routing test — both aliases". Sending it is the only
end-to-end delivery proof, and I cannot send on your behalf.

### The Attention feature was deliberately left out

The other session's work is still in the working tree and was **excluded from this
deploy**. I stripped it from `dist/` only and never touched the source files, so that
session's work is untouched and rebuilding restores it.

Confirmed not live: `/app/attention.js` and `/app/attention.css` both 404, and the
built `index.html` contains zero references to them. The Attention chip does not
appear in the strip.

One thing did ship: the `GET /api/attention` endpoint that session added to
`worker/index.js`, along with `data/attention.json`. Removing it would have meant
operating on a file another session was actively editing, which is the riskier move.
I read it first. It is read-only, cached six hours, writes nothing but its own
`attention:live:v2` key, and cannot collide with intake submissions. It is live but
unreferenced, so no visitor reaches it.

**To deploy Attention**, from a clean tree with that session finished:

```bash
cd output/aschcapital
python3 ingest/build_site.py     # restores attention.js/css into dist
./deploy/deploy.sh
```

No stripping step. That is the only difference.

## Reading what people send

Submissions land in Workers KV. `scripts/aschcapital/intake.py` is the way in and out:

```
python3 scripts/aschcapital/intake.py list              everything, newest first
python3 scripts/aschcapital/intake.py list recruiting   one kind
python3 scripts/aschcapital/intake.py list --days 7     the last week
python3 scripts/aschcapital/intake.py show A4EC8811     one submission in full
python3 scripts/aschcapital/intake.py fetch A4EC8811    download it, files and all
python3 scripts/aschcapital/intake.py delete A4EC8811   permanently remove it
```

`delete` exists because the Trading DNA page promises "ask us to delete it at any point
and we will", and a promise with no mechanism behind it is just copy.

Optional: `npx wrangler secret put NOTIFY_WEBHOOK` gives you a one-line ping per
submission so you are not polling. It never carries the uploaded file.

## Storage: KV now, R2 later if you want

R2 needs a dashboard opt-in that has not happened, and enabling it means accepting R2's
terms, which is yours to do rather than mine. Rather than let that hold up the launch,
the Worker takes R2 whenever an R2 bucket is bound and uses Workers KV otherwise. KV caps
a value at 25 MiB, which was already the per-file limit, so nothing that would have fit
in R2 gets turned away today.

If you ever want to move (R2 is the better home for blobs and cheaper at volume):

    1. dash.cloudflare.com -> R2 -> enable
    2. npx wrangler r2 bucket create asch-intake
    3. add the r2_buckets block in wrangler.jsonc, comma after kv_namespaces
    4. ./deploy/deploy.sh

The key layout is identical in both, so nothing else changes.

### Still unreviewed

The recruiting page publishes performance thresholds and compensation terms I drafted.
They are live. They have not been through you line by line or through counsel.

---

## What was added

Four chips at the right-hand end of the existing strip, in this order:

| Chip | Route | What it is |
|---|---|---|
| Prometheus | `#/prometheus` | Two assessments plus the combined profile |
| Recruiting | `#/recruiting` | Track record submission for managers |
| AI Operations | `#/ai-operations` | The AI practice, with checkout |
| Trading DNA | `#/trading-dna` | Trade history intake, report returned by hand |

New files, all additive:

```
app/bank-personality.js   100 items, scoring, 9 bands, narrative library, 8 trait interactions
app/bank-ai.js             40 items, 8 modes, 3 tiers, 6 task targets, 6 archetypes, exercises
app/offers.js              the offer ladder and its prices  <- needs your sign-off
app/labs.js                all four sections, the assessment runner, forms
app/labs.css               shared styles for the four sections
app/skins.css              the four per-section identities
app/art.js                 nine generated SVG pieces, two of them data-driven
worker/index.js            three intake endpoints, files to R2
```

### Each section has its own design

Clicking a chip changes `data-skin` on `<html>` and the whole page changes with it:
background, type, colour, chrome, panels, buttons, forms. The index screens carry no
skin, so nothing in `app/skins.css` can reach them. Fonts are only fetched the first
time a skin is actually opened.

| Section | Look | Drawn from |
|---|---|---|
| Prometheus | Midnight field, mint and violet accents, geometric grotesque set tight, soft radial light, rounded panels | the modern dark AI product genre (ref: aimodes.ai) |
| Recruiting | Cream stock, bold transitional serif at display size, hairline rules, wide-tracked claret labels, italic captions | the print-editorial fund letter genre (ref: militiacapital.com) |
| AI Operations | Near-black instrument panel on a blueprint grid, machined corner ticks, cyan status lights, heavy condensed caps, monospace chrome | nothing. Original |
| Trading DNA | Near-white, one enormous centred headline, very long vertical rhythm, borderless cards on soft grey, a single blue, one full-bleed black break | the large-format product page genre (ref: apple.com) |

**On trade dress.** Each skin takes the *conventions* of a genre, which nobody owns, and
then uses its own type, palette, spacing and layout. No logo, wordmark, icon, image or
line of copy is taken from any reference site, and no typeface or palette is reused from
one. Prometheus uses Plus Jakarta Sans and a mint-and-violet pair, not Geist and a single
teal. Recruiting uses Source Serif 4 and a browner claret, not PT Serif and a dusty rose.
Trading DNA uses its own greys and its own blue. AI Operations has no reference at all.
The reference sites were inspected for their grammar only, and that inspection is
recorded in the header of `app/skins.css`.

### The artwork

Nine pieces, all generated SVG in `app/art.js`. No stock photography, nothing fetched
from anywhere: no licensing to track, no third-party host in the critical path, no layout
shift, sharp at any size, and each piece paints with the same CSS custom properties as
the rest of the page, so it re-colours itself per skin automatically. Total cost is 25 KB
of code for all nine.

| Section | Piece | Drawn from |
|---|---|---|
| Prometheus | Two clusters converging on one point | idea, not data |
| Prometheus | Ten-axis aspect radar | **your actual percentiles** |
| Prometheus | Eight-axis mode radar, you against target | **your actual distribution** |
| Recruiting | 100 positions collapsing into 6 bets | fixed illustration |
| Recruiting | What The Tape reads: drawdown shape, the few days that carried it | fixed illustration, labelled as such |
| AI Operations | Impact against effort, with the start-here quadrant | fixed illustration |
| AI Operations | 14 steps, 3 removed, 1 skill | fixed illustration |
| Trading DNA | The helix, six markers riding the front strand | decorative |
| Trading DNA | Hold asymmetry, winners against losers | fixed illustration |

The two radars are the ones worth pointing at. They are computed from real results, so
the shape moves when the answers do, and the mode radar shows the misalignment far more
directly than the bar chart underneath it: the filled shape bulges toward Oracle while
the dashed target points at Verification.

Two carry an explicit on-image disclaimer because they could otherwise be misread as a
performance claim: The Tape schematic says "illustrative, not a track record, not ours,
not anyone's", and the hold-asymmetry bars say "illustrative shape, your report uses your
own fills".

Dense diagrams scroll inside their own box below 620px rather than shrinking their labels
into illegibility. No page overflows horizontally at 390px.

`index.html` changed in six places and nothing else: two stylesheet links, four script
tags, and four edits inside `route()` so it can hand a route to the new module, keep
the chip highlighted on sub-routes, and render a new screen without waiting on the
1.7 MB index fetch. Every pre-existing screen was re-tested after the change.

---

## Blocking, in order of how blocking it is

Resolved 2026-08-24: contact address, prices, Trading DNA pricing.

### 1. Only one thing is still genuinely blocking: three Stripe Payment Links

Everything else can ship today. Until the links exist, the three paid buttons on
AI Operations read "Request an invoice" and post to the enquiry endpoint, which is a
working path, just a slower one. See "How buying works" below.

### 2. The recruiting terms are drafted, not agreed

Not blocking the build, blocking your comfort. The Sharpe and alpha thresholds, the
$1M-$10M starting allocation, 15% over a hurdle, and the three-year partnership are
written in the fund-letter shape with Asch numbers. They are mine, not yours. Published
compensation terms and performance thresholds for a fund are not ordinary web copy, and
this page probably wants a look from counsel as well as from you. A standard
"not an offer" disclosure is already on the page.

### 3. Email, resolved with a note

Both addresses now build to `dave@cravecookies.com`, set in `ingest/build_site.py`.

That includes `corrections@aschcapital.com`, which was already live on the Disclosures
screen and was also dead. It is now your Crave mailbox too. If you would rather keep
that one on the Asch domain, change `CONTACT` back once mail exists there; `ENQUIRIES`
is the one the four new forms use and they are independent.

### Settled

| | |
|---|---|
| The Fifteen | free |
| The Operations Assessment | $1,500 one time |
| Operator retainer | $1,750 per month |
| Embedded retainer | $3,000 per month |
| Trading DNA | free intake, no price, revisit if submissions come in |

---

## How buying works, in full

Checkout is a URL. Whatever you take money through, the button links to it, so no payment
secret lives in the Worker and no card or bank detail touches aschcapital.com. Set the
provider in `app/offers.js`:

```js
const PAYMENT = {
  provider: "venmo",              // "venmo" | "stripe" | "none"
  venmo: { handle: "", business: false }
};
```

### Venmo, which is what you asked for

It works, and it is already built and tested. Give me the handle and it goes live in one
redeploy. The button deep-links to Venmo with the amount and a note pre-filled, for
example `venmo.com/<handle>?txn=pay&amount=1500&note=Asch Capital The Operations Assessment`.

Three things you should know before we switch it on:

1. **It has to be a Venmo business profile.** Venmo's user agreement does not permit
   taking payment for goods and services through a personal account. The penalty is
   frozen funds and a closed account, and finding that out with a client's $3,000 sitting
   in it is the bad version. A business profile is legitimate for exactly this, and costs
   1.9% + $0.10 per transaction.
2. **Venmo cannot bill on a schedule.** There is no subscription product. The button pays
   the first month and every month after that is a request you send by hand. Two retainer
   clients means two manual requests a month forever. The card copy says this out loud
   rather than letting a buyer assume it renews, and each retainer card also carries a
   "Rather be invoiced" button.
3. **US only**, and the payer needs a Venmo account.

My recommendation, which is yours to overrule: Venmo for The Fifteen and the one-time
$1,500 assessment, where it is a genuinely good fit and removes all friction. For the
$1,750 and $3,000 monthly retainers, a real processor pays for itself in the first month
of not chasing anyone. There is also a positioning question in asking the owner of a $10M
business to Venmo you three thousand dollars, and that one is entirely your call.

### Stripe, if you ever want it

Set `provider: "stripe"`, create three Payment Links in the dashboard (one-time $1,500;
recurring monthly $1,750; recurring monthly $3,000), and paste each URL into the matching
`link:` field. Recurring prices then actually recur. Everything else stays the same.

---|---|---|
| The Operations Assessment | $1,500 | one time |
| Operator | $1,750 | recurring, monthly |
| Embedded | $3,000 | recurring, monthly |

Then paste each URL into the matching empty `link: ""` in `app/offers.js` and rebuild.
The button turns into a real checkout link the moment a URL is there; I have tested that
switchover. I cannot create the Stripe account or the products myself, because that
means handling your payment credentials.

**What you give up.** No coupon codes, no upsells, no self-serve cancellation portal
unless you switch the billing portal on separately in Stripe, which is a dashboard
setting and not a code change. If you later want any of that, the swap to Checkout
Sessions touches one file.

---

## Rebuilding and redeploying

```bash
cd output/aschcapital
python3 ingest/build_site.py
./deploy/deploy.sh
```

---

## Reading submissions

They land in R2 as `<kind>/<YYYY-MM-DD>/<REF>/meta.json` plus the uploaded files.

```bash
npx wrangler r2 object list asch-intake --prefix recruiting/
npx wrangler r2 object get asch-intake recruiting/2026-08-24/F7C33E33/meta.json
```

Set `NOTIFY_WEBHOOK` and you get a one-line ping per submission instead of polling. The
ping never carries the uploaded file.

---

## What I deliberately did not do

- **No Prometheus results are sent anywhere.** Scoring is arithmetic and runs in the
  browser. There is no endpoint for results, no account, and no model call. The export
  and delete buttons on the hub operate on local storage.
- **No transcript ingestion.** `FR-10`–`FR-13` want a model call per submitted
  conversation. The self-report instrument ships instead and every report screen says
  which one it is. The scoring spine is shared, so the transcript version slots in.
- **No broker connection anywhere near Trading DNA.** It reads a file a person chose to
  upload. No credential, no API, no order path — the trading locks in `CLAUDE.md` are
  kept clear by never going near them.
- **Nothing deployed.** The Prometheus write-lock in `docs/rules/prometheus.md` is still
  `PENDING DEFINITION`, which says to draft and wait. Pushing this live is your call,
  and that lock still needs a real definition written into `CLAUDE.md`.

---

## Honesty notes that are load-bearing, not disclaimers

Three claims on these pages are weaker than they look, and the copy says so on the page
rather than in a footer. If you soften any of this, the pages start overclaiming.

- **Percentiles are not against a population.** Prometheus has no normative sample
  (`RISK-02`). Percentiles are computed against a declared theoretical distribution
  (`P_NORM` in `app/bank-personality.js`). Replace that one object when real
  respondents exist and every band moves with it.
- **The AI target mixes are priors.** Reasoned from what each kind of work requires, not
  fitted to outcome data, because there is none yet (`FR-15` requires them to be
  independently derived, and they are).
- **The correlated layer is hypotheses, not findings.** No dataset connects the two
  instruments yet. Population benchmarking is switched off on that page rather than
  shown against a sample too small to mean anything, which is the graceful degradation
  `FR-24` asks for.

---

## Verified before handover

- All 100 personality items present, 10 per aspect, 5 keyed each way
- All 40 AI items present, 5 per mode; all six task target rows sum to 100 and cover all 8 modes
- Both assessments completed end to end, both reports and the combined profile render
- Every pre-existing screen re-tested: index, methodology, ceiling, by size, by state,
  screener, disclosures, a firm page, unknown-route fallback, and nav search
- Chip highlighting correct on sub-routes; titles correct on all screens
- Form validation: empty, bad email, oversize file, honeypot
- Worker tested under `wrangler dev`: asset passthrough byte-for-byte, method and route
  guards, all three endpoints, files and metadata written to R2, IP hashed not stored
- A real browser submission on the Trading DNA form landed in the bucket intact
- Mobile at 390px: assessment runner, both reports, all four sections
- All four skins apply on entry and come back off on exit, including `theme-color`
- No horizontal overflow on any skinned screen at 390px
- All 17 routes render with no console errors after skinning
- Radar geometry checked numerically: every label sits on its own axis within 2 degrees
- No artwork label clips its viewBox on either radar
- Every piece of art lays out with real height on every route; none scroll the page wide
