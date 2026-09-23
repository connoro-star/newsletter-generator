# Newsletter HTML Builder

Single-file tool that turns a form into email-safe newsletter HTML for pasting
into the CMS. No build step, no server, no dependencies.

## Use it

**https://connoro-star.github.io/newsletter-generator/**

Or open `index.html` in a browser (double-click, or drag into a tab) - it is a
single file and behaves the same either way.

> The hosted copy is **public and unauthenticated**: free GitHub Pages offers no
> access control, so anyone with the link can use it. It is an internal tool
> published on a public host, not a product. Do not put anything confidential
> into a draft. See [SECURITY.md](SECURITY.md).

The preview renders at a 600px email width with no gutter of its own: blocks are
sized to the newsletter itself, so they run edge to edge and what you see is the
full column.

## Alignment

Every top-level block renders through `shell()` at the same **Block width**
(Settings, default 600px), centred, so all section types share one column with
identical left and right edges.

The rule that keeps them aligned: **padding and borders go on the inner `<td>`,
never on the `<table>`.** CSS width is content-box, so a `<table width="100%">`
carrying 15px padding and a 3px border renders 36px wider than its siblings.
That is how the four sections originally drifted apart:

| Block | Was | Now |
|---|---|---|
| Featured Article | 596px (100% + padding + 3px border) | 560px |
| Article List | 562px (100% + 1px border) | 560px |
| Deal Grid | 560px fixed, cards inset 4px | 560px, cards flush |
| Link List | client-default `<ul>` indent | 560px, bullets flush |

Block-level tables still declare a `width` attribute beside `max-width`, which
costs nothing in modern clients and holds the newsletter to its column in
Outlook. Images do not: they are percentage-only. The Link List is built from
table rows rather
than a `<ul>` because every email client applies its own list indent and
Outlook's cannot be overridden.

Changing Block width rescales everything together; only the deal cards still
derive a pixel width, at `(width - 8) / 2` for the card itself.

Every element inside a block is a percentage, images included, so they reflow
with whatever column they are given rather than being pinned to the width they
were generated at.

In the Article List the image is top-aligned with the headline rather than
centred against the row, so a one-line and a three-line headline both start
level with the top of the thumbnail.

Every bordered card keeps a 6px gutter between its content and its border - the
featured tile, the article-list rows and the deal cards. At the default 600px
block width an article-list row gives 586px of its 599px to content. The deal
cards keep a taller vertical rhythm (16px above the title, 12px under the
price); only the horizontal padding is 6px.

The blocks themselves have no outer gutter: at a 600px block width they span the
600px newsletter, so the cards' borders sit on its edges.

The 12px gutter between an article-list thumbnail and its headline stays - that
is spacing between two elements, not between an element and the border.

## Blocks

| Block | Output |
|---|---|
| Featured Article | Bordered hero card, entirely clickable: headline, dek, image, summary, Read More pill |
| Section Heading | `Today's Hits` / `MOST READ THIS WEEK` / `CURATED DEALS` style H2 |
| Article List | Thumbnail at 40% of the tile + title + subtitle rows; starts with 5 empty tiles |
| Deal Grid | Product cards, 2 per row, auto `SAVE x%` badge |
| Link List | Numbered rows of linked headlines, marker in the brand accent |
| Disclaimer | Italic affiliate footnote |
| Custom HTML | Raw markup, passed through untouched |

Each section type has its own colour, shown on the add button, the card's left
edge, the dot in its header, and its type label - so the shape of an issue reads
at a glance while scrolling:

| | Section | | Section |
|---|---|---|---|
| amber | Featured Article | rose | Link List |
| violet | Heading | slate | Disclaimer |
| sky | Article List | fuchsia | Custom HTML |
| emerald | Deal Grid | | |

Drag block headers to reorder. Arrows reorder items inside a block.

Vertical spacing is not a block you manage. The generator inserts a blank
paragraph:

- above every section heading (except a heading that opens the newsletter)
- after every deal grid
- above a link list that is **not** introduced by its own heading - a link list
  running straight on from a deal grid gets its own gap, while one sitting under
  a `MOST READ THIS WEEK` heading stays tight to that heading

Inside the featured tile everything sits on one 6px rhythm: the padding, the gap
under the headline, the dek, the hero image and the summary paragraph. The gap no
longer opens out when a headline has no sub-headline - it used to go to 16px.
The tile's 16px outer margin is unchanged, so the space between the tile and the
next block is the same as before.

## Mobile and dark mode

The output starts with a small `<style>` block. Everything in it is an override
on a design that already works without it - the inline styles are still the
source of truth - because it covers the one thing inline CSS cannot express:
`prefers-color-scheme`. The thumbnail sizing is inline, so it survives a CMS
that strips `<style>`.

**The article-list thumbnail is 40% of the tile**, not a pixel size. It was a
fixed 200px, which on a handset left the headline almost no room. A percentage
holds the same proportion at every width, so there is no breakpoint to drift out
of tune and no media query needed for it at all:

| Viewport | Cell | Image | Text | Share |
|---|---|---|---|---|
| 640px desktop | 214px | 202px | 320px | 40% |
| 430px Pro Max | 152px | 140px | 228px | 40% |
| 390px iPhone 14 | 136px | 124px | 204px | 40% |
| 375px 13 mini | 130px | 118px | 195px | 40% |
| 320px SE | 108px | 96px | 162px | 40% |

Images carry no pixel width at all - `width:100%` inside a percentage cell, at
every block. Outlook's Word engine ignores percentage widths on an image and
will fall back to the file's intrinsic size there; that is an accepted
trade-off rather than something to design around.

**Dark mode follows the reader.** `prefers-color-scheme: dark` repaints the card
surfaces, borders and text through the `nl-card` / `nl-strong` / `nl-body` /
`nl-muted` classes. Use the **Dark** and **Mobile** toggles above the preview to
check both without a dark-mode mail client.

Three caveats worth knowing:

- **Outlook desktop ignores `<style>` entirely.** It gets the inline design:
  full-size thumbnails, light mode. That is the intended fallback, not a bug.
- **Gmail ignores `prefers-color-scheme`** and applies its own inversion. These
  colours are chosen to survive that rather than fight it.
- **Some CMSes strip `<style>` on paste.** If dark mode and the mobile
  thumbnails stop working after pasting, that is what happened - check the
  source view for the block, and paste it into the template head instead.

Product shots with white backgrounds still read as white blocks in dark mode.
That is the source image, not the CSS.

## Auto-fill

Paste a URL, click **Auto-fill**: the page is fetched through `r.jina.ai` and its
Open Graph tags supply title, dek, and image. Amazon pages have no OG tags, so
product URLs are read with Amazon-specific selectors that also pull the sale
price and list price.

**Auto-fill replaces every field it owns on that item**, rather than only
filling blanks. Pointing an item at a new URL and pressing Auto-fill therefore
clears what the previous URL left behind: if the new page has no price, the old
price is emptied rather than sitting next to a new title as though it belonged
with it. The URL box itself is the input and is never touched.

| Block | Fields Auto-fill owns |
|---|---|
| Featured Article | headline, dek, summary, hero image, image alt |
| Article List | title, subtitle, thumbnail, image alt |
| Link List | title |
| Deal Grid | title, image, image alt, price, list price |

> This reverses the old behaviour, where a filled field was left alone. Anything
> you have written by hand into one of those fields will be overwritten the next
> time you press Auto-fill on that item.

Always check what came back before sending - scraping is best-effort.

### How the Summary Paragraph is chosen

The summary is the article's first real paragraph, which means stepping over the
author card. A bio or author excerpt reads like prose and is easily long enough
to pass a length test, so it used to win on document order and land in the
Featured Article's summary.

The paragraph is now chosen structurally rather than by wording, because a bio
has no reliable phrasing to match on:

- Paragraphs inside anything whose class or id contains `author`, `byline`,
  `bio`, `excerpt`, `contributor` or `profile` are skipped, as are `[rel=author]`,
  `<address>`, `<aside>`, `<footer>`, `<nav>`, `<header>`, `<figcaption>` and
  `<blockquote>`.
- A marked-up article body wins over document order, so a long paragraph in a
  sidebar or a related-articles rail cannot be picked first. The scope is
  `<article>`, `[itemprop=articleBody]`, `article-body`, `entry-content`,
  `<main>`.
- If that finds nothing the whole page is tried, and failing that the summary
  falls back to the page's meta description.

Add a container to the `NOT_BODY` list in `index.html` if a site puts its author
card somewhere new.

Keyless use of the reader is rate limited to about **20 fetches per minute**. If
you hit it the toast says so; wait a minute and carry on. `api.allorigins.win` is
tried as a fallback. Failures name the actual cause, with detail in the browser
console.

> The previous `corsproxy.io` endpoint was retired - it now returns
> `403 keyless_legacy_url` and needs a paid API key. That is what broke
> auto-fill. `newsletter_generator_project/script.js` still calls it and is
> broken the same way.

### Amazon blocks the readers, so Deal Grid auto-fill does not work

Checked 2026-09-23. Amazon answers the reader with a bot-check interstitial -
about 3.5KB of "Click the button below to continue shopping" - rather than the
product page, consistently across different product URLs. There is no product
title, price or image in it to read.

That page is a `200` with real markup, so it used to pass every check: the
scrape ran against the interstitial, found no `#productTitle`, fell back to the
document title and filled the product name with **"Amazon.com"** while the toast
said it had succeeded. A challenge page is now detected and treated as a failed
fetch, so the next reader is tried and, if that also fails, auto-fill says so
and fills nothing. Wrong data is worse than no data.

`api.allorigins.win`, the fallback, is currently returning `522` for every URL,
Amazon or not - it is down, not just blocked.

**So: paste deal titles, prices and images by hand for now.** Auto-fill still
works for the Featured Article, Article List and Link List, which read editorial
pages rather than Amazon.

Restoring it means a reader Amazon does not block - a paid tier, or Amazon's own
Product Advertising API, which needs credentials and an approved Associates
account. Adding another free proxy sends every pasted URL to one more third
party, so that is a decision to take deliberately rather than by default.

## Featured Article

The whole tile is one link to the Article URL - headline, dek, image, summary and
button are all inside a single `<a>`. There is no separate link field; leave the
Article URL empty and the tile renders as plain unlinked content.

Two consequences worth knowing before editing `gFeatured`:

- **Nothing inside the tile may be an anchor of its own.** Nested `<a>` is invalid
  and the HTML parser breaks it apart, so the headline, image and Read More button
  are plain elements that inherit the click from the wrapper. If you add a second
  link inside the tile, the markup will silently come apart.
- **Every text colour is stated inline** (`#000000` headline, `#333333` body, accent
  on the pill). Inside an `<a>`, anything left to inherit turns default link-blue
  and underlined in clients that ignore `inherit`. This is verified against hostile
  link styling; keep it explicit.

The padding sits on the link rather than the cell so the click target covers the
whole bordered tile - measured at 99.3% coverage, the remainder being the 1px
border. In Outlook's Word rendering engine, block-level anchors are unreliable:
expect the text and image to stay clickable there, but not necessarily the empty
space between them.

Byline fields (author name, date, author page, author photo) were removed. If you
need a byline back, it lived in `gFeatured` - see git history or add a Custom HTML
block.

## Sites

The **Site** selector fills in that site's accent colour, Amazon affiliate tag
and the publication named in the affiliate disclaimer:

| Site | Accent | Affiliate tag | Disclaimer names |
|---|---|---|---|
| AP - Android Police | `#e01a4f` | `ap-newsletter04-20` | Android Police |
| MUO - MakeUseOf | `#c70016` | `mak0954-20` | MakeUseOf |
| PL - Pocket-lint | `#f01e25` | `pl-newsletter-20` | Pocket-lint |

The accent recolours the byline link, the Read More pill and the GET DEAL
buttons together. Both fields stay editable after picking a site; editing either
to something that is not a preset flips the selector to **Custom**, and typing a
preset's values back re-detects it. Projects saved before sites existed get
theirs inferred from the colour and tag they already carry.

Add a site by adding one entry to the `SITES` map in `index.html`. The `name` is
what the disclaimer uses, so a new site needs nothing else.

### How the disclaimer follows the site

Switching site rewrites the disclaimer **only if it still carries some preset's
exact wording**. A disclaimer you have reworded is your text and is left alone -
matching against every preset's wording is what tells the two apart.

- A new Disclaimer block takes the site currently selected.
- A project saved before this worked - a PL issue still naming Android Police -
  is corrected to its own site when it loads.
- Under **Custom** there is no brand, so a new disclaimer reads `[site name]`
  rather than borrowing another publication's. Switching *to* Custom leaves
  existing wording alone.

## Deals

- Enter sale price and list price; the `SAVE x%` badge is calculated for you.
  Type a value in the Badge field to override it.
- Amazon links get the affiliate tag from Settings applied automatically
  (replacing any existing `tag=`). Non-Amazon links are left alone.

## Saving

- **Copy HTML for CMS** (`Cmd+Shift+C`) — clipboard, ready for the source view.
- **Save JSON** (`Cmd+S`) — the editable project. Keep these to reuse an issue.
- **Import JSON** — reload a saved project.
- **Download .html** — the generated output as a file.

Work is also autosaved to this browser's local storage, so a refresh will not
lose the issue in progress. Local storage is per-browser and per-machine:
use Save JSON for anything you need to keep or hand to someone else.

## Starting an issue

**New issue** loads the house layout, empty. Use it to start each week.

It exists because the autosaved draft always wins at boot - that is what keeps
an issue in progress across a refresh, but it also means a change to the house
layout never reaches anyone who has opened the tool before, because their saved
blocks are restored instead. **New issue** is how you get the current layout
without clearing site data.

The layout, which is also what a brand-new session opens on:

1. Featured Article
2. Section Heading - `Today's Hits`
3. Article List - 5 empty rows
4. Section Heading - `MOST READ THIS WEEK`
5. Link List - 5 empty rows
6. Section Heading - `CURATED DEALS`
7. Deal Grid
8. Affiliate Disclaimer

**Clear** empties the fields, not the layout. Every block keeps its type, its
position and its row count; section headings keep their text, and boilerplate
that comes from a block's defaults - the CTA labels, the disclaimer wording -
comes back. To remove a block, use the x on the block itself.

**Load sample** replaces the issue with a filled-in example, for checking how
the output looks.

> **Clear changed meaning.** It used to delete every block. It now empties the
> fields and leaves the layout alone. If you relied on it to start from nothing,
> remove the blocks you do not want with their own x.

Clear cannot be undone and it overwrites the autosave, so use **Save JSON**
first for anything you need to keep. The autosave keeps one state per browser -
there is no version history.

## Editing the template

### Renaming a block type

`normalise` drops any block whose `type` it does not recognise, silently - so
renaming a type key destroys every saved draft and every previously exported
JSON that used the old name. `TYPE_ALIASES` maps old keys to current ones and
must be extended whenever a type is renamed. `icymi -> mostread` is there as the
worked example.

### Where the markup lives

All markup lives in the `gFeatured` / `gHits` / `gDeals` / `gMostRead` functions in
`index.html`. They emit the approved markup verbatim, so a style change there
changes every newsletter. The featured and deal-card output is diff-verified
byte-for-byte against the approved mock-up.
