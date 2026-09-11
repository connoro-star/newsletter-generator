# Newsletter HTML Builder

Single-file tool that turns a form into email-safe newsletter HTML for pasting
into the CMS. No build step, no server, no dependencies.

## Use it

Open `index.html` in a browser (double-click, or drag into a tab).

The preview renders at a 600px email width with 20px padding, leaving a 560px
content column - so what you see matches a real inbox.

## Alignment

Every top-level block renders through `shell()` at the same **Block width**
(Settings, default 560px), centred, so all section types share one column with
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

Widths are declared twice on purpose - the `width` attribute for Outlook,
`max-width` for everything else. The Link List is built from table rows rather
than a `<ul>` because every email client applies its own list indent and
Outlook's cannot be overridden.

Changing Block width rescales everything together: the featured card's inner
column becomes `width - 32`, and each deal card `(width - 8) / 2`.

## Blocks

| Block | Output |
|---|---|
| Featured Article | Bordered hero card, entirely clickable: headline, dek, image, summary, Read More pill |
| Section Heading | `Today's Hits` / `CURATED DEALS` / `ICYMI` style H2 |
| Article List | 200px thumbnail + title + subtitle rows; starts with 5 empty tiles |
| Deal Grid | Product cards, 2 per row, auto `SAVE x%` badge |
| Link List | Plain `<ul>` of linked headlines |
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
  an `ICYMI` heading stays tight to that heading

Inside the featured tile, the sub-headline sits 6px under the headline so the two
read as a pair; a headline with no sub-headline keeps the full 16px gap to
whatever follows.

## Auto-fill

Paste a URL, click **Auto-fill**: the page is fetched through `r.jina.ai` and its
Open Graph tags supply title, dek, and image. Amazon pages have no OG tags, so
product URLs are read with Amazon-specific selectors that also pull the sale
price and list price.

Only fields you have left blank get filled, so your edits are never overwritten.
Always check what came back before sending - scraping is best-effort.

Keyless use of the reader is rate limited to about **20 fetches per minute**. If
you hit it the toast says so; wait a minute and carry on. `api.allorigins.win` is
tried as a fallback. Failures name the actual cause, with detail in the browser
console.

> The previous `corsproxy.io` endpoint was retired - it now returns
> `403 keyless_legacy_url` and needs a paid API key. That is what broke
> auto-fill. `newsletter_generator_project/script.js` still calls it and is
> broken the same way.

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

## Editing the template

All markup lives in the `gFeatured` / `gHits` / `gDeals` / `gIcymi` functions in
`index.html`. They emit the approved markup verbatim, so a style change there
changes every newsletter. The featured and deal-card output is diff-verified
byte-for-byte against the approved mock-up.
