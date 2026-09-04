# modernmarvel.io

Source for the two Cloudflare Workers that serve modernmarvel.io.

| Folder    | Worker              | Serves                       | Assets          |
|-----------|---------------------|------------------------------|-----------------|
| `home/`   | `modernmarvel-home` | `modernmarvel.io/` (root)    | `home/public`   |
| `digest/` | `modernmarvel-io`   | everything else on the apex  | `digest/public` |

Both are **assets-only** Workers. Neither runs JavaScript at request time —
Cloudflare serves the files under each `public/` directory directly. There is
no `src/index.js` in either folder because there is no server code to put in
one.

## How the two Workers divide one domain

This is the part worth understanding before changing anything.

`modernmarvel-io` holds a **Custom Domain** on `modernmarvel.io`. A Custom
Domain claims the entire hostname, so by default that Worker answers every
path. `modernmarvel-home` then wins back exactly one URL — the bare root —
with a more specific route, `modernmarvel.io/`.

    modernmarvel.io/               -> modernmarvel-home  (specific route wins)
    modernmarvel.io/ai-digest*     -> modernmarvel-io    (specific route)
    modernmarvel.io/anything-else  -> modernmarvel-io    (Custom Domain catch-all)

A consequence worth knowing: an unrecognised URL such as
`modernmarvel.io/nonsense` returns the **digest's** 404 page, because the
digest Worker is what catches it. That is current behaviour, not a bug, but it
is surprising if you assume the landing page owns the domain.

### Why the config files must list routes that already exist

Wrangler treats each config file as the source of truth for routing and
reconciles the account to match it on deploy. A route or Custom Domain that is
live but **absent from the config can be removed by a routine deploy.**

The apex Custom Domain is the dangerous case. It owns the DNS record for
`modernmarvel.io`. If a deploy removes it, the apex stops resolving — which
strands `modernmarvel-home` too, because its route then has no DNS record to
attach to. One ordinary-looking `wrangler deploy` in `digest/` would take down
both pages.

Both live entries are therefore declared in `digest/wrangler.jsonc`. Do not
delete either one to "clean up" the file.

### The one edit that breaks everything

Do not widen either pattern to `modernmarvel.io/*`. Doing so out-specifies the
`modernmarvel.io/` route and takes the landing page dark. This is the
highest-risk edit in the repo, and it is the one thing a preview URL will not
catch, because previews run on `workers.dev` rather than on the custom domain.

## Known gap: www

`www.modernmarvel.io` does not resolve — it returns NXDOMAIN. The Custom Domain
for it described in the old publish notes was never created. Nothing depends on
it today; it is recorded here so it is a decision rather than a surprise.

## Layout

    home/
      wrangler.toml                       settings for modernmarvel-home
      public/index.html                   the landing page
    digest/
      wrangler.jsonc                      settings for modernmarvel-io
      public/404.html                     shown for any unmatched path on the apex
      public/ai-digest/index.html         the current edition
      public/ai-digest/about.html         how the digest is put together
      public/ai-digest/archive/index.html the list of all editions
      public/ai-digest/archive/2026-09-01.html  edition 001, kept permanently
      public/ai-digest/share-draft.md     the share note for edition 001

## Status

Phase 1: source under version control, matching what is deployed.

Phase 2 (not yet done): connect each Worker to this repo via Workers Builds so
that merging to `main` deploys and branches produce preview URLs. Until then,
deploys are manual. **Committing here does not publish anything.**

## Deploying

See `DEPLOYING.md`.
