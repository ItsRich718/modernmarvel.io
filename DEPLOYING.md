# Deploying

Deploys are manual and per-folder. Nothing in this repo publishes itself, and
pushing to GitHub changes nothing on the live site.

## One-time setup

You need Wrangler, the tool Cloudflare uses to upload a site. In a terminal,
run these three lines one at a time:

    npm config set allow-scripts=esbuild,workerd --location=user
    npm install -g wrangler
    wrangler login

The last one opens your browser to sign in to Cloudflare. You never need to
repeat any of this.

**Why the first line.** Recent npm versions refuse, by default, to let a
package run setup commands during install, because those setup commands have
been the most common way malicious packages attack a developer's machine.
Wrangler depends on `esbuild` and `workerd`, whose setup step downloads the
right build for your machine, so they genuinely need the permission. That line
grants it to those two packages only and leaves the protection in place for
everything else.

Note that the setting holds a list and setting it again *replaces* the list
rather than adding to it. If a future tool needs the same permission, include
the old names alongside the new one. To see what is currently allowed:

    npm config get allow-scripts

There is also a blanket option permitting every package to run setup commands.
It is named `--dangerously-allow-all-scripts`, and the name is the advice.

## Check it locally first

Every page is self-contained, so you can open any of them straight from disk
with no server and no internet:

- `home/public/index.html`
- `digest/public/ai-digest/index.html`

Click through the Archive and About links and confirm they resolve.

## Deploy

Move into the folder for whichever Worker you changed, and run one command.

For the landing page:

    cd home
    wrangler deploy

For the digest:

    cd digest
    wrangler deploy

Wrangler reads the config file in the current directory, so the folder you are
in decides which Worker you publish. If it complains that it cannot find a
configuration file, you are in the wrong folder.

Deploy only the folder you changed. There is no reason to publish both.

## After deploying

Wrangler prints a `workers.dev` address. Open it and confirm the change is
there. Then check the real domain in a private window — `modernmarvel.io/` or
`modernmarvel.io/ai-digest` — because the custom-domain route is the part the
preview address cannot verify.

## Publishing a future edition

Replace the contents of `digest/public/ai-digest/`, leave the dated file in
`archive/` alone, add the new dated copy alongside it, then deploy `digest`.
Once an edition is published it is permanent and is never edited again.

## Routes — read before your first deploy

Every route and Custom Domain that is live in Cloudflare is declared in the
config files. That is deliberate and it must stay that way: Wrangler reconciles
the account to match the config on each deploy, so an entry that is live but
missing from the file can be deleted by a routine deploy.

The entry that matters most is the Custom Domain on `modernmarvel.io` in
`digest/wrangler.jsonc`. It owns the DNS record for the apex. Losing it takes
down the landing page as well as the digest. See the README for the full
explanation.

Do not widen either pattern to `modernmarvel.io/*`.

If you need to confirm what is actually live, open each Worker in the Cloudflare
dashboard under **Settings → Domains & Routes**. While the site is working,
Cloudflare is correct by definition — change the config file to match it, not
the other way around, and commit that change.

---

# Workers Builds (CI)

These settings live in the Cloudflare dashboard, not in this repo. They are
recorded here so that the configuration is written down somewhere you can
diff, which is the same reason the routes are declared in the config files.

Each Worker gets its own connection to this one repository, distinguished by
its root directory.

## Settings

| Setting                    | `modernmarvel-home` | `modernmarvel-io`   |
|----------------------------|---------------------|---------------------|
| Repository                 | `ItsRich718/modernmarvel.io` | same       |
| Production branch          | `main`              | `main`              |
| **Root directory**         | `home`              | `digest`            |
| Build command              | *(leave empty)*     | *(leave empty)*     |
| Deploy command             | `npx wrangler deploy` | `npx wrangler deploy` |
| Non-prod deploy command    | `npx wrangler versions upload` | same     |
| Build watch paths: include | `home/*`            | `digest/*`          |

The build command is empty on purpose. Both Workers are assets-only, there is
nothing to compile, and there is no `package.json` in this repository.

**Root directory is the setting that makes this work.** There is no config
file at the repository root, so a build that runs from the root will fail to
find one. Each connection must point at its own folder.

**Watch paths matter more than they look.** Without them, every push rebuilds
and redeploys both Workers, so editing the landing page would trigger a
deploy of the digest — and a digest deploy is the one that reconciles the
apex Custom Domain. Scope each Worker to its own folder so unrelated commits
cannot touch it.

## Wrangler version

`npx wrangler deploy` with no `package.json` resolves to whatever the latest
Wrangler is on the day the build runs. Known-good version as of this writing
is **4.128.0**, which is what the config files were validated against.

To pin it, set the deploy commands to `npx wrangler@4.128.0 deploy` and
`npx wrangler@4.128.0 versions upload`. This trades automatic updates for
reproducible builds, and it has to be bumped by hand. Given that a deploy
reconciles live routing, reproducible is the safer default.

## Order of connection

Connect `modernmarvel-home` first. Its route is an ordinary route that owns no
DNS record, so a bad reconciliation there is recoverable.

Connect `modernmarvel-io` second, and only after a deploy of it has been seen
to succeed. Connecting starts a build straight away, and that build runs
`wrangler deploy`, which reconciles the apex Custom Domain. CI cannot answer a
prompt if Wrangler asks one.

## Verifying the first build

The repository was seeded from the folders the site was already deployed
from, so the first production build of each Worker should change nothing that
is visible. After each one:

1. Confirm the build succeeded in **Settings** > **Builds**.
2. Load `modernmarvel.io/` and `modernmarvel.io/ai-digest` in a private window.
3. Load `modernmarvel.io/nonsense` and confirm it still returns the digest's
   404 page. This is the check that proves the apex Custom Domain survived,
   and it is the one most easily forgotten.

If step 3 fails, the Custom Domain was dropped. Re-add it in the dashboard
under **modernmarvel-io** > **Settings** > **Domains & Routes** before doing
anything else.
