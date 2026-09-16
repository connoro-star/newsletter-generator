# Deploying

`index.html` is a single static file with no build step. Any static host serves
it as-is: **build command empty, output directory `/`.**

Auto-fill keeps working when hosted. `r.jina.ai` echoes the `Origin` header, so
a hosted HTTPS page is allowed exactly as a local `file://` page is - verified
against both `*.pages.dev` and `*.github.io` origins.

## GitHub Pages (current host)

The repo is **public** - free GitHub Pages only serves public repos, and the
affiliate tags in the `SITES` map are public with it. That was a deliberate
call; see "Security trade-off" below before assuming otherwise.

```bash
gh repo edit connoro-star/newsletter-generator --visibility public --accept-visibility-change-consequences
gh api repos/connoro-star/newsletter-generator/pages -X POST -f 'source[branch]=main' -f 'source[path]=/'
# -> https://connoro-star.github.io/newsletter-generator/
```

`git push` to `main` redeploys; a build takes roughly a minute. Check it with:

```bash
curl -s https://connoro-star.github.io/newsletter-generator/ | grep -o '<title>[^<]*</title>'
# expected: <title>Newsletter HTML Builder</title>
```

`.nojekyll` stops GitHub running the site through Jekyll. Without it Jekyll
would try to interpret Liquid tags (`{{`, `{%`) inside `index.html` and would
hide underscore-prefixed files like `_headers` from the build.

### Security trade-off versus Cloudflare

GitHub Pages sends **none** of the headers in `_headers` - verified against live
`*.github.io` origins. `_headers` is inert here and kept only for a move back to
Cloudflare or Netlify. The policy now travels in `index.html` as a
`<meta http-equiv="Content-Security-Policy">`, which covers everything except:

| Lost on GitHub Pages | Effect |
|---|---|
| `frame-ancestors` / `X-Frame-Options` | the page can be framed by any site - clickjacking is not blocked |
| `Permissions-Policy` | camera/mic/geolocation are not pre-denied (the app requests none) |
| `X-Content-Type-Options: nosniff` | relies on GitHub serving correct content types, which it does |

`Referrer-Policy` survives as `<meta name="referrer" content="no-referrer">`.

If any of that matters later, move back to Cloudflare Pages via **Connect to
Git** - `_headers` starts applying again with no code change.

## Netlify

Connect the repo, same build settings. Free tier gives a public URL; site-wide
password protection is a paid plan. `_headers` is honoured.

## If the live URL serves an old version

Always confirm what the host is actually serving before debugging the app. On
2026-09-16 `newsletter-generator.pages.dev` served a 12 KB `Newsletter Generator`
v1.7 file with none of this repo's markup (no `addbar`, no `data-add`, no
`SITES`) while `origin/main` held the correct 56 KB builder. The cause: that
Pages project had been created by **direct upload**, so it had no link to the
repo and pushes never triggered a build - silently, with no error anywhere.
Nothing was wrong with the code or the CSP. Cloudflare was abandoned for
GitHub Pages rather than reconnected.

The same check applies to any host:

```bash
curl -s <the live url> | grep -o '<title>[^<]*</title>'
# expected: <title>Newsletter HTML Builder</title>
```

If that prints something else, the problem is the deployment, not the app.

## The site is public and unauthenticated

Anyone with the URL can read the page source, including the affiliate tags in
the `SITES` map. Free GitHub Pages has no access control of any kind. If the
site ever needs restricting, that means leaving GitHub Pages - Cloudflare Pages
plus an Access policy on a custom subdomain is the documented route.

## After deploying

`git push` to `main` redeploys.

Drafts autosave per browser, so each person has their own in-progress issue.
To hand an issue to someone else use **Save JSON** -> they **Import JSON**.

Tell people the auto-fill rule when you share the link: pasting a URL sends it
to `r.jina.ai`, so it must not be used on unpublished or staging URLs.
