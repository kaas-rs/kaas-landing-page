# kaas-landing-page

The landing page at **<https://kaas.rs/>** — the animated hero, the blurb, and
the cards that lead into the kaas book.

This repo owns the site: GitHub Pages and the `kaas.rs` custom domain are
configured **here**, not on [kaas-rs/kaas](https://github.com/kaas-rs/kaas).

## How the site is assembled

The published site is two pieces from two repos:

| Path on the site | Comes from |
|---|---|
| `/` | `index.html` in this repo |
| `/book/…` | the mdbook in `kaas-rs/kaas` under `docs/`, built there |

kaas builds its own book (it needs the repo's `mdbook` pins, the mermaid and
linkcheck backends, and the `cargo xtask check-docs-drift` gates that keep the
compatibility tables honest) and force-pushes the rendered HTML to its
`book-dist` branch. `.github/workflows/publish.yml` here checks that branch
out, mounts it under `book/`, and deploys the pair to Pages.

```
kaas-rs/kaas ──mdbook build──▶ book-dist branch ─┐
                                                 ├─▶ _site/ ──▶ Pages (kaas.rs)
kaas-rs/kaas-landing-page ── index.html ─────────┘
```

So the site rebuilds on either half changing:

- a push to `main` here,
- a `book-updated` repository dispatch, sent by kaas's *Docs Publish* workflow
  once it has refreshed `book-dist`,
- a daily cron, as a fallback for the dispatch not arriving,
- or a manual *Run workflow* on *Publish site*.

Before deploying, the workflow re-checks every `href="book/…"` on the landing
page against the book it just pulled — a card pointing at a page that got
renamed or removed on the kaas side fails the build instead of shipping dead.

## Editing

`index.html` is standalone: inline CSS and JS, a `data:` URI favicon, no build
step and no external requests. Open it in a browser to preview.

Its `book/…` links are relative, so they resolve on the Pages preview URL
(`kaas-rs.github.io/kaas-landing-page/`) as well as on `kaas.rs` — but only
after a deploy has mounted the book. Opening the file straight off disk gives
you the page without a working `/book/`.

## The cross-repo trigger

kaas needs a token to dispatch to this repo — the default `GITHUB_TOKEN` is
scoped to its own repo and cannot. kaas's *Docs Publish* workflow reads it from
a `LANDING_DISPATCH_TOKEN` secret and **skips the dispatch when that secret is
absent**, so an unconfigured token costs you a stale site, never a red build.
To (re)configure it: create a fine-grained PAT scoped to this repo with
*Contents: read and write* (what `POST /repos/…/dispatches` requires), then run

```bash
gh secret set LANDING_DISPATCH_TOKEN --repo kaas-rs/kaas
```

Without it, a book update reaches the site via the daily cron — up to a day
late — or immediately on a manual run of *Publish site*. Note GitHub disables
scheduled workflows in a repo with no pushes for 60 days, which this repo can
easily hit; the token is what keeps the site prompt.

## License

Apache-2.0, same as kaas.
