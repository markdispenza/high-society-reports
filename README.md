# High Society — local ranking & directory-listing investigation (report pages)

Live: **https://markdispenza.github.io/high-society-reports/**

The 15 rendered report pages from a local-SEO investigation run 2026-09-02 → 2026-09-03 on
High Society Weed Dispensary New Buffalo, its competitors, and 29 client locations.

## What this repo is — and is not

This is a **presentation build**: the finished, human-readable report pages only. Each page is a
single self-contained HTML file with its own inline stylesheet — no build step, no server, no
external assets. Open any of them straight from disk.

It is **not** the investigation repository. The pipeline code, the raw evidence captures, the
per-listing proof pages, the data files, the agent briefs and the engineer handoff package live in
the private source repo and are deliberately not published here.

It is also **not** any business's website. Every business named in these pages — the subject, its
competitors, the directory vendors — is discussed as the subject of an audit.

## Indexation posture — read before "fixing" anything

Every page carries:

```html
<meta name="robots" content="noindex, follow">
```

and `robots.txt` says `Allow: /`. That pairing looks backwards. It is deliberate:

1. **`noindex` alone — never with a cross-domain `rel=canonical`.** They are contradictory signals,
   and Google's documented failure mode is that the noindex gets inherited by the canonical target.
   If that target were a real business's live site, this build would be aiming a deindex signal at
   their actual pages. No canonical tags are emitted here.

2. **`follow`, not `nofollow`.** `nofollow` stops a crawler following links, so a crawler entering
   at the landing page would read *its* noindex and stop — never fetching the other 15 pages, never
   seeing *their* noindex. Already-known URLs would persist in the index. `follow` is what lets the
   noindex on every page actually be read.

3. **`robots.txt` must allow the crawl.** `Disallow: /` blocks the fetch that would reveal the
   noindex, and a blocked-but-linked URL can still be indexed URL-only, indefinitely. Allowing the
   crawl is what actually achieves de-indexation.

   Caveat worth knowing: this is a **project-subpath** Pages site, and `robots.txt` is only honoured
   at the host root (`markdispenza.github.io/robots.txt`), which this repo does not control. So the
   `robots.txt` here is documentation, not the mechanism. **The per-page meta tag is the only
   operative control. Do not remove it.**

`sitemap.xml` is published at the subpath for first-party crawlers (Screaming Frog list mode, which
needs it because a `nofollow`-free flat site is still easier to enumerate from a list). It is
deliberately **not** advertised via a `Sitemap:` directive in `robots.txt` — advertising a sitemap
for a noindexed build is a contradictory "please index" signal.

None of the above is access control. This repo is public; treat the URLs as shareable-with-anyone.

## Redactions

Agency-internal material is stripped at **build** time, not by hand-editing the output, so a
rebuild cannot quietly reinstate it. Three classes are removed:

| Redacted | Shown as | Why |
|---|---|---|
| Staging hostnames (`seogstage.com`, `*.staging.sgen.com`) | `[agency-staging-host]` | Reachable internal infrastructure; publishing the naming pattern is an attack-surface hint |
| Local Falcon credit balance | `[redacted]` | Account state — no analytical value |
| Vendor spend: totals, subtotals, budget caps | `$[redacted]` | Agency cost structure |

Analysis is unaffected. Relative claims survive redaction — *"any Google operator multiplies the
SERP price by 5"* keeps its force without the absolute figures — so the methodology arguments still
stand on their own.

**What is deliberately *not* redacted:** the dollar figures in `reports/citation-audit.html` are
scraped third-party evidence, not agency spend — dispensary menu prices (`2 for $140`,
`$80.00 Ounce`), a directory promo (`nd $60, get $25 off`), and job-listing wages
(`$15.00 Per Hour`) lifted from SERP snippets. Blanket-redacting them would destroy the evidence the
audit rests on. That file is exempt from the *blanket* money rule only — its agency cost tiles are
still redacted, matched on the tile **label** rather than the amount.

`high-society-verify.mjs` fails the build if any of these reappear, and the exempt file is checked
too rather than skipped.

## Pages

| Page | What |
|---|---|
| `FINDINGS.html` | The investigation's conclusions |
| `PLAYBOOK.html` | How local ranking actually works, as measured |
| `ACTION-PLAN.html` | Gap and task list, ranked |
| `DIRECTORY-LISTINGS.html` | Directory-listing matrix with verification status |
| `RULES-TO-AVOID.html` | Tactics the evidence says not to use |
| `reports/competitor-hub.html` | Competitor gap analysis hub |
| `reports/ranking-signals.html` | Which signals move local rank |
| `reports/client-portfolio.html` | Defect audit across 29 client locations |
| `reports/market-leaders.html` | Market leader head-to-head |
| `reports/market-leaders-new-buffalo.html` | The same, New Buffalo only |
| `reports/keyword-match.html` | Keyword-in-name vs local rank |
| `reports/category-match.html` | Primary category vs rank |
| `reports/proximity-model.html` | How much of local rank is distance |
| `reports/scale-test.html` | Citations vs local rank, at scale |
| `reports/citation-audit.html` | Citation audit with proof status |

## Rebuilding

The site is generated from the private source repo, not edited by hand. The generators live in the
workspace beside the source checkout (`Code/`), not in this repo, because they read the private tree:

```bash
node high-society-build.mjs           # copy pages, inject noindex, emit robots.txt + sitemap.xml
node high-society-landing.mjs         # regenerate index.html from the page summaries
node high-society-verify.mjs          # fail-closed gate on the built bytes
node high-society-verify.mjs --live   # the same gate, run against the deployed URLs
```

Editing a page in this repo by hand will be overwritten on the next build. Change the source.

The verifier fails the build if any page loses its `noindex`, gains a `nofollow`, gains a
`rel=canonical`, carries two robots tags, puts the tag outside the head, or is orphaned from the
landing page.
