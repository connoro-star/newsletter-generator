# Deploying

`index.html` is a single static file with no build step. Any static host serves
it as-is: **build command empty, output directory `/`.**

Auto-fill keeps working when hosted. `r.jina.ai` echoes the `Origin` header, so
a hosted HTTPS page is allowed exactly as a local `file://` page is - verified
against both `*.pages.dev` and `*.github.io` origins.

## Cloudflare Pages + Access (recommended)

Repo stays private, the URL is restricted to people you name, free.

1. dash.cloudflare.com -> Workers & Pages -> Create -> Pages -> **Connect to Git**
2. Authorise GitHub for **this repository only**, pick `newsletter-generator`
3. Build settings - framework preset **None**, build command **empty**, output
   directory **`/`**
4. Save and Deploy -> `newsletter-generator.pages.dev`
5. Restrict it: Zero Trust -> Access -> Applications -> **Add a self-hosted
   application**, pointed at that hostname, with an Allow policy for
   *emails ending in* your company domain

Free for up to 50 users. Note: Access policies attach most reliably to a custom
domain you have in Cloudflare. If you have no domain there, protecting the bare
`.pages.dev` host may not be offered - in that case use the GitHub Pages route
below and scrub the tags, or add a custom subdomain first.

## GitHub Pages (public)

Simplest, but the site is public even if the repo is private, and on a free
plan Pages requires a public repo. **Blank the affiliate tag defaults in the
`SITES` map before doing this** - they are revenue-attributing identifiers.

```bash
gh repo edit connoro-star/newsletter-generator --visibility public --accept-visibility-change-consequences
gh api repos/connoro-star/newsletter-generator/pages -X POST -f 'source[branch]=main' -f 'source[path]=/'
# -> https://connoro-star.github.io/newsletter-generator/
```

GitHub Pages ignores `_headers`; it serves its own fixed header set.

## Netlify

Connect the repo, same build settings. Free tier gives a public URL; site-wide
password protection is a paid plan. `_headers` is honoured.

## After deploying

`git push` redeploys automatically on Cloudflare and Netlify.

Drafts autosave per browser, so each person has their own in-progress issue.
To hand an issue to someone else use **Save JSON** -> they **Import JSON**.

Tell people the auto-fill rule when you share the link: pasting a URL sends it
to `r.jina.ai`, so it must not be used on unpublished or staging URLs.
