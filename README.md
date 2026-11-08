# WorkflowPilot Lab

English-language affiliate content site for freelancers and solo business owners,
published with GitHub Pages at <https://nexus441014-sketch.github.io/>.

**This repository is the live website.** Every committed file is served at
`https://nexus441014-sketch.github.io/<path>`. The build inputs, the scripts and the
monitoring data are deliberately kept out of the repository (see `.gitignore`) so
they are not published.

## Write an article

Create one Markdown file in `content/`:

```
---
title: The headline readers see as the <h1>
seo_title: Shorter title for the browser tab and search results (optional)
description: One sentence, 50-160 characters. Becomes the meta description.
slug: url-path-without-the-html
tag: Email marketing
summary: The standfirst paragraph under the headline.
published: 2026-11-08
updated: 2026-11-08
weight: 40
affiliate: no
related: other-slug, another-slug
---

## A section heading

Normal paragraphs. [Links](https://example.com/) are given
rel="noopener noreferrer" automatically when they point off-site.

- bullet lists
1. numbered lists

| Table | Columns |
| --- | --- |
| work | too |

> A blockquote becomes a highlighted notice box.
```

`weight` controls the order on the home page and the guides page (lower comes first).
`affiliate: yes` switches the disclosure box to the wording used when the article
actually contains affiliate links. `related` picks the "Keep reading" links; if it is
left out, three other articles are chosen automatically.

Then run `./publish.sh`.

## The one command

```
./publish.sh              # build, check, and show what would change. Changes nothing.
./publish.sh --commit     # also commit locally. Still not published.
./publish.sh --push       # commit and publish to GitHub Pages.
./publish.sh --external   # also request every outbound link (add to any of the above)
```

Nothing is committed unless the build and all checks pass, and nothing is published
unless `--push` is passed. The default run is a dry run.

### First push: GitHub authentication

This Mac has no GitHub credentials stored, so the very first `--push` has to be done
by hand once. GitHub does not accept account passwords over HTTPS — it needs a
personal access token:

1. Create a token at <https://github.com/settings/tokens> (Tokens (classic) → Generate
   new token, scope `repo`), or a fine-grained token with Contents: Read and write on
   `nexus441014-sketch.github.io`. Copy it; it is shown only once.
2. Run `git push origin HEAD` in this folder. When prompted, enter the GitHub username
   as the username and paste the token as the password.

`credential.helper` is set to `osxkeychain` for this repository, so the token is stored
in the macOS Keychain and `./publish.sh --push` works unattended from then on.

### What the build generates

`tools/build.py` writes, every run:

- `articles/<slug>.html` for each file in `content/`, with canonical URL, Open Graph
  tags, `Article` JSON-LD, a table of contents for longer pieces, an affiliate
  disclosure box above the body, and "Keep reading" internal links
- `index.html` and `guides.html`
- `sitemap.xml` and a `robots.txt` that points to it
- the shared navigation and the canonical/Open Graph tags on `about.html`,
  `contact.html`, `privacy.html` and `disclosure.html`

Generated files are never edited by hand: the next build overwrites them. Edit
`content/` instead.

### What the checks cover

`tools/check.py` fails the publish on any of these:

- an internal link or `#anchor` that points at something that does not exist
- a missing `<title>`, missing meta description, or more than one `<h1>`
- a duplicate title or duplicate meta description across pages
- an article with no affiliate disclosure block
- a `sitemap.xml` entry whose file is missing
- with `--external`, an outbound link that does not return a 2xx or 3xx

It warns, without failing, about over-long titles and descriptions and missing
canonical tags. Search-engine verification files such as
`googleff35d75fec088486.html` are skipped and must be left exactly as issued.

## Affiliate terms monitor

```
python3 tools/monitor_offers.py          # fetch, diff against the last snapshot, write a report
python3 tools/monitor_offers.py --init   # take a first snapshot without reporting changes
python3 tools/monitor_offers.py --quiet  # report only the sources that changed
```

`offers/sources.json` lists the official affiliate and pricing pages to watch.
Each run stores the page's visible text in `offers/snapshots/` and writes
`reports/offers-<date>.md`, which highlights changed lines containing figures
(percentages, amounts, cookie windows) and commercial keywords.

It reads public pages only. It never logs in, applies, or sends anything.

Pages that block scripted requests are reported as unreadable rather than assumed
unchanged — currently `make.com`, which returns HTTP 403 and has to be opened in a
browser by hand.

**The rule the monitor exists to enforce:** a commission rate, cookie window or
payout threshold goes into an article only if it can be read on the provider's own
page on the day of writing, with the date recorded. Anything else stays out.

## Editorial rules

`CLAUDE.md` holds them. The ones that matter most: no invented prices, free-plan
limits, or first-hand testing; no claimed affiliate relationship that does not exist;
disclosure near the links; verify from the provider's own pages.

## Requirements

Python 3 (standard library only, works on the system Python 3.8), `git`, and `curl`.
No dependencies to install, no paid services.

`curl` does the HTTP work rather than Python's `urllib`, because the bundled Python
on this machine does not trust the certificate store used on this network.
