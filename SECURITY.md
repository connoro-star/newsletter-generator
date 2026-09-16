# Security

This is an **internal editorial tool**, published on a public host. It is not a
product, it carries no warranty, and it is not maintained on a security SLA.

## What it is

A single static HTML file that turns a form into email-safe newsletter markup.
It has no backend, no accounts, no database and no server-side code. Everything
runs in the visitor's browser.

## What that means in practice

**The hosted copy is public and unauthenticated.** Free GitHub Pages offers no
access control of any kind, so anyone with the link can open and use it. There
is nothing to log into and nothing to break into.

**Drafts are stored in the browser, unencrypted.** Work autosaves to
`localStorage`, which is per-browser and per-machine. It is never transmitted
anywhere. Two consequences:

- `*.github.io` is a shared origin, so storage is shared with anything else
  published under that domain. Treat drafts as non-confidential.
- Clearing site data loses the draft. Use **Save JSON** for anything that
  matters.

**Do not put confidential or unpublished material into a draft.**

**Auto-fill sends URLs to a third party.** Pasting a URL and pressing Auto-fill
sends that URL to `r.jina.ai` to scrape the title, image, author and date. Do
not use it on unpublished, embargoed or staging URLs - the URL leaves your
network.

**Affiliate tags are public.** The Amazon tags in the `SITES` map ship in the
page source and are visible in the public repository. This is a known and
accepted trade-off of hosting on free GitHub Pages. They are attribution
identifiers, not credentials: they grant no account access.

**Custom HTML is passed through untouched.** That block exists to emit raw
markup and does not sanitise it. Everything else is escaped. Treat markup pasted
into it the way you would treat pasted code - the trust boundary is whoever has
access to the tool.

## Headers

`_headers` is inert on GitHub Pages, which sends none of it. The policy travels
in `index.html` as a `<meta http-equiv="Content-Security-Policy">`. Two
directives have no meta equivalent and are therefore not in force:
`frame-ancestors` (so the page can be framed) and `Permissions-Policy`. See
DEPLOY.md for the full trade-off and how to get them back.

## Reporting something

Open an issue on the repository, or contact the maintainer directly if the
problem is better not described in public.
