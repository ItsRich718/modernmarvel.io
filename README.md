# modernmarvel.io

Source for the two Cloudflare Workers that serve modernmarvel.io.

| Folder    | Worker              | Serves                       | Assets          |
|-----------|---------------------|------------------------------|-----------------|
| `home/`   | `modernmarvel-home` | `modernmarvel.io/` (root)    | `home/public`   |
| `digest/` | `modernmarvel-io`   | `modernmarvel.io/ai-digest*` | `digest/public` |

Both are **assets-only** Workers. Neither runs JavaScript at request time —
Cloudflare serves the files under each `public/` directory directly. There is
no `src/index.js` in either folder because there is no server code to put in
one.

## The routes are the fragile part

Two Workers share one domain, and they stay out of each other's way only
because their routes are scoped narrowly:

- `modernmarvel-home` is bound to `modernmarvel.io/` — the trailing slash and
  the *absence* of a wildcard are both deliberate.
- `modernmarvel-io` is bound to `modernmarvel.io/ai-digest*`.

Widening either pattern to `modernmarvel.io/*` takes the other one dark. This
is the highest-risk edit in the repo, and it is the one thing a preview URL
will not catch, because previews run on `workers.dev` rather than on the
custom domain.

## Layout

    home/
      wrangler.toml                       settings for modernmarvel-home
      public/index.html                   the landing page
    digest/
      wrangler.jsonc                      settings for modernmarvel-io
      public/404.html                     shown for an unmatched path
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
