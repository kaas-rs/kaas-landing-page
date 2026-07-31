# kaas-landing-page

The landing page at **<https://kaas.rs/>** — the animated hero, the blurb, and
the cards that lead into the kaas book.

This repo owns that site: GitHub Pages and the `kaas.rs` custom domain are
configured here.

## What this repo is, and isn't

It is one file. `index.html` is standalone — inline CSS and JS, a `data:` URI
favicon, no build step and no external requests. Open it in a browser to
preview; what you see is what deploys. `.github/workflows/publish.yml` copies
it into `_site/` and hands that to Pages.

It is **not** the documentation. The kaas book is a separate Pages site, built
and deployed from [kaas-rs/kaas](https://github.com/kaas-rs/kaas) at
<https://kaas-rs.github.io/kaas/>, and the cards here link to it by absolute
URL.

```
kaas-rs/kaas              ──▶ kaas-rs.github.io/kaas/   (the book)
kaas-rs/kaas-landing-page ──▶ kaas.rs                   (this page)  ──links──▶
```

The two sites share no build, no branch, and no credential. That is
deliberate, and it is worth knowing why, because the obvious "nicer" layout —
`kaas.rs/` for this page and `kaas.rs/book/` for the book — was built first and
then removed. GitHub Pages binds a domain to exactly one repo, so serving both
halves under `kaas.rs` means one repo must hand its output to the other: an
orphan branch carrying the render, a cross-repo checkout, a PAT to trigger the
redeploy, and a cron to cover that PAT expiring. All of that machinery bought
one URL shape. Two sites need none of it.

## The one thing that spans both repos

These cards deep-link to `index.html` and `getting-started.html` in the book.
Because those links are absolute and cross-origin, nothing here can detect a
rename on the kaas side — the link would simply start 404ing. So kaas's CI
asserts both pages keep building, and that check is the only guard on it. If
you add a card pointing at a new book page, add it to that check too
(the `docs` job in kaas's `.github/workflows/ci.yml`).

## License

Apache-2.0, same as kaas.
