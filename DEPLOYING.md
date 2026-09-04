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

## Routes

The routes are already configured in Cloudflare and are declared in each
folder's config file. Do not widen either one — see the README for why. If you
ever need to confirm what is actually live, open the Worker in the Cloudflare
dashboard under **Settings → Domains & Routes** and compare. Cloudflare is
correct by definition while the site is working; change the config file to
match it, not the other way around.
