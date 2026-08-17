# Paginated Table — Tableau Viz Extension

A worksheet viz extension that renders your data as a **paginated table** instead of one long
scrolling crosstab. Built to replace a pagination workaround that used
30 duplicate dashboards plus `Page Size` / `Current Page` parameters and 31 `Page N | Last Page`
helper sheets.

## What it does

- **Fixed rows per page** with a full control bar: `« ‹ 1 2 3 … N › »`, `Page X of Y`,
  a go-to-page box, a rows-per-page dropdown, and a `1–25 of 412 rows` counter.
- **Group header bands** driven by a dimension on the `Group By` shelf (e.g. Superintendent).
  When a group spans a page break the band repeats, marked *continued*.
- **Quick filter** box (client-side, across all displayed columns), plus **CSV** and **PDF**
  download of every filtered row — not just the visible page.
- **Date-labelled headers**: a labelled column reads `Week 3` over `Jul 18–24`, computed from the
  report's through-date parameter and re-labelled automatically when that parameter changes. Target
  columns by name prefix, or tick individual measure columns and set each one's period by hand —
  the period drives both the heading and the dates.
- **Merged header bands**: a second header tier above the columns — `JOB INFO` spanning three
  columns, `WEEKLY CHANGE` spanning the week columns — optionally stacked with the **overall date
  range** those columns cover (`Jul 11 – Aug 7`). That aggregate span can also sit under a single
  chosen column such as `Total Change`. Carried into the PDF and CSV too.
- **Column divider lines** at every band boundary, running the full height of the table.
- **Adjustable gridline weight**, 0-4px, matched in the PDF.
- **Conditional formatting rules** — negatives bold red, thresholds, text matches — applied per
  column or across all numeric columns, and carried into the PDF.
- **Nested sorting** — an ordered list of sort levels, defaulting to the Group By value ascending so
  the bands stay contiguous. Click a header to sort within the groups, shift-click to nest another
  level. Numbers sort numerically and dates chronologically, not as text.
- **Freeze the first N columns** so job identifiers stay visible while scrolling sideways.
- **Per-column widths, font sizes and alignment**, with wrapping and rows that grow to fit.
- **Rename or hide any column's header** from a per-column picker.
- **Null handling** — show empty values as blank, `0`, an en dash, or your own text.
- Keyboard paging: `←` `→`, `Page Up` / `Page Down`, `Home`, `End`.
- Cell text uses Tableau's own **formatted values**, so number, date, and percent formats defined
  in the workbook carry through.
- Full formatting control (fonts, colors, padding, banding, header labels) in a settings dialog
  behind the ⚙ button. Settings are stored in the workbook, so viewers see your configuration.

## Files

| File | Purpose |
|---|---|
| `index.html` | The visualization. Vanilla JS + CSS, no external libraries. |
| `settings-dialog.html` | Formatting / pagination dialog opened by the ⚙ button. |
| `paginated-table.trex` | Manifest for the hosted (GitHub Pages) build. |
| `paginated-table-localhost.trex` | Manifest for local testing on `http://localhost:8765`. |
| `tableau.extensions.1.latest.min.js` | The Extensions API (v1.16). **Required** — must sit next to `index.html`. |
| `jspdf.umd.min.js` | jsPDF 2.5.2. Needed only for PDF export; loaded on first click. |
| `jspdf.plugin.autotable.min.js` | jspdf-autotable 3.8.4, the table renderer for the PDF. |
| `.nojekyll` | Stops GitHub Pages from processing the folder with Jekyll. |

The two jsPDF files are fetched on demand the first time someone clicks **PDF**, not at page load:
400KB of blocking script would push `initializeAsync` past Tableau's handshake timeout. Ship them
alongside `index.html` — if they're missing, the PDF button reports it and everything else keeps
working.

No D3 or CDN chart library is used — a table needs plain DOM — so nothing is blocked by
Tableau Desktop's embedded browser. The Extensions API is loaded from the bundled same-origin copy
with a plain `<script>` tag, the same way the official Tableau samples do it: Desktop's embedded
browser sometimes blocks `extensions.tableausoftware.com`, and a slow or hanging CDN request delays
`initializeAsync` past Tableau's handshake timeout. Keep that `.js` file alongside `index.html`
wherever you host it.

Requires **Tableau Desktop / Server / Cloud 2024.2 or later** (viz extensions, API 1.11).

## 1. Test locally first

```bash
cd paginated-table-viz
python3 -m http.server 8765
```

Open <http://localhost:8765/index.html> in a browser — it renders with demo data so you can preview
the layout. Append settings as query parameters to try combinations, e.g.
`?rowsPerPage=15&freezeCols=2&showRowNumbers=true&barPosition=top`.

Then in Tableau Desktop, follow *Build the worksheet* below using
`paginated-table-localhost.trex`. Keep the server running while you test.

## 2. Build the worksheet

1. Duplicate `Overview Table` (or start a new sheet) and put the fields you want on **Detail**,
   at whatever level of granularity should produce one table row.
2. On the **Marks card**, open the mark type dropdown and choose **Add Extension…**, then select
   the `.trex` file. The marks card now shows two shelves: **Columns** and **Group By**.
3. Drag fields onto **Columns** in the left-to-right order you want them displayed (up to 30).
   Any field type is accepted — strings, numbers, dates, booleans, discrete or continuous.
   For the Overview report that's `Col 1`…`Col 21` — job/plan, customer, location/template,
   duration, framing start, CN comp date and days, FQI, total change, Week 1–7, POO, and the
   two schedule-start dates.
4. Drag `Superintendent` onto **Group By** to get the grouped bands.
5. Click ⚙ in the control bar and set rows per page, header labels, fonts, and colors.

## 3. Deploy to GitHub Pages

The hosted build lives at:

```
https://lcristaldi-raulen.github.io/paginated-table-viz/index.html
```

That URL is already written into `paginated-table.trex`, so the manifest needs no editing as long as
the repo is named `paginated-table-viz` and owned by that account. If either changes, edit the
`<url>` in the manifest to match — Tableau fetches exactly that address and nothing else.

1. Create a **public** repo named `paginated-table-viz` and commit every file in this folder.
   `jspdf.umd.min.js` and `jspdf.plugin.autotable.min.js` are only needed for PDF export, but
   without them the **⤓ PDF** button reports the libraries as missing.
2. **Settings → Pages → Deploy from a branch → `main` → `/ (root)`**. The first build takes a
   minute or two; Pages shows the live URL when it's done.
3. Open the URL in a browser before pointing Tableau at it. You should see the table with the
   yellow *demo data — not connected to Tableau* chip, which is the correct standalone behaviour.

Pages must be public. A private Pages site requires a paid plan and makes every viewer log into
GitHub before the extension will load, which is not workable inside a dashboard.

### Tableau Cloud / Server allowlist

Under **Settings → Extensions → Add Extension**, add the **full URL including `/index.html`**:

```
https://lcristaldi-raulen.github.io/paginated-table-viz/index.html
```

Set it to **allow full data access** — the extension reads summary data, and the ⚙ dialog needs to
save settings into the workbook.

## 4. Swap it into the Overview dashboard

Replace the `Overview Table` object on the *Overview* dashboard with the new sheet. Once it's in,
these can all be retired:

- dashboards `1`–`30`
- worksheets `Page 1 | Last Page` … `Page 30 | Last Page`, `Page All | Last Page`
- worksheets `Overview Table (21)`…`(30)` and `Table (1)`…`(20)`
- the `Page Size`, `Current Page`, `Page Number`, and `Last Page` calculations and parameters

The `Headers` sheet can go too — the extension draws both tiers itself now: merged bands over the
column names (⚙ → Headers → Merged bands), with the names coming from the fields or your header
overrides.

## Header text

**⚙ → Headers → Header text** lists every column with a box for its heading and a **Hide** tick.
Leave a name blank to keep Tableau's field name. Names are stored per *field*, so renaming survives
column reordering — and a heading containing a comma is fine, which the old positional list couldn't
manage.

**Hide** blanks that column's header cell. The column and its data stay put; only the heading text
goes — which is what you want under a merged band that already names the group, or for a column
whose contents speak for themselves. Hiding also drops the date sub-label and blanks the heading in
the PDF and CSV exports. **Hide all** / **Show all** flip every column at once.

A rename beats the automatic `Week {n}` period label, so you can override a single column's heading
without turning that feature off.

Settings saved before this picker existed used a comma-separated positional list; those are migrated
into the per-column map the first time you open the dialog, and still apply until then.

## Merged header bands

**⚙ → Headers → Merged bands** adds a second header tier above the column names — the two-tier
block the `Headers` worksheet draws today, but built into the table so it scrolls, exports, and
stays aligned automatically.

Type the same band name next to each column that belongs together and they merge into one spanning
heading. Only *adjacent* columns merge, so two separate runs sharing a name stay as two bands rather
than one impossible span. Columns you leave blank simply have no band above them. A summary line
under the picker shows what you'll get — `JOB INFO (3 cols) · WEEKLY CHANGE (4 cols)` — before you
apply.

### Overall date range on a band

**Add the overall date range to bands covering dated columns** puts the full span those columns
cover onto the band itself — the earliest start to the latest end across every dated column in the
span. A `WEEKLY CHANGE` band over Week 1–4 with a through-date of 8/7 reads:

```
            WEEKLY CHANGE
             Jul 11 – Aug 7
  Week 1     Week 2     Week 3     Week 4
  Aug 1–7    Jul 25–31  Jul 18–24  Jul 11–17
```

`Range position` stacks it under the band name or puts it in brackets after it. It follows the same
date format as the column labels and recomputes with the parameter, so it never drifts out of sync
with the columns underneath.

Columns that carry a date label but no band name group into a range band of their own, so you can
get the overall span without naming a band at all. Bands with no dated columns — `JOB INFO` — are
left alone.

### Overall date range on a single column

The picker's **All** column puts that same aggregate span under one specific header instead of on a
band. Tick **All** next to `Total Change` and it reads:

```
                FINAL QI PROJECTED END DATE
                       Jul 11 – Aug 7
  Total Change    Week 1     Week 2      Week 3      Week 4
  Jul 11 – Aug 7  Aug 1–7    Jul 25–31   Jul 18–24   Jul 11–17
```

A column marked **All** keeps its own name — it stands for the whole span rather than one period,
so the `Week {n}` rename deliberately doesn't apply to it. Its period box is disabled, and it's
excluded from the min/max calculation, so it can never skew the aggregate it's displaying. The
picker previews the resolved span next to the tick.

This lives in the per-column picker, so switch **Apply to** to *Columns I pick below* to use it —
**Auto-number from names** reproduces prefix-mode numbering in one click if that's where you
started.

Both header tiers stay pinned when you scroll, band colours are on the Colors tab, and the band text
can be centred over its columns or left-aligned. The bands repeat on every PDF page along with the
column headers. CSV gets them as a leading row with each label in the first cell of its run, since
merged cells don't exist in CSV.

One alignment note: if you also freeze columns, a band is only pinned when its whole span sits
inside the frozen range. A band that straddles the boundary scrolls with the rest of the table —
so set `Freeze first N columns` to a band boundary if you want the two to move together.

### Bands over selected columns only

**Show the band only over columns that have one** (on by default) gives every un-banded column the
full height of both header tiers, so its heading uses the whole block rather than sitting below an
empty strip. The band tier then exists only above the columns you actually named — in the Overview
report, `WEEKLY CHANGE` over the week columns while `Job | Plan`, `Cus` and the rest run full height
beside it. This applies to the PDF as well as the view.

Untick it to go back to every column having a band cell, empty or not.

**Top-align a heading that spans both tiers** (on by default) then decides where in that merged
block the heading sits. On, it sits at the top, level with the band names beside it, which is what
makes a two-tier header read as one row of headings rather than two staggered ones. Off, it follows
the **Header vertical** setting like every other heading — so untick this and set that to *Middle*
if you prefer them floating in the centre of the block.

### Band width

**Size the band to the sum of its columns** (on by default) makes a merged band exactly as wide as
the columns beneath it, trimming a long band name with an ellipsis rather than letting it stretch
those columns. A spanning header cell normally takes part in column sizing, so without this a name
like `FINAL QI PROJECTED END DATE` widens four 32px week columns to 54px each. Turn it off to get
that older behaviour, where the band name wins and the columns give way.

The full name stays available on hover.

The band row carries one pixel of height beyond what the row padding gives it — 0.75pt in the PDF —
which keeps a merged name clear of the rules above and below it.

### Wrapping a long band name

**Wrap a long band name onto more lines** (on by default) breaks a name that won't fit onto as many
lines as it needs and grows the band row to suit, instead of trimming it with an ellipsis. The
columns underneath keep exactly the widths you set — the band label is measured outside the normal
table flow, so a wrapped name adds height and never width. Every band cell takes the same height, so
the tier stays level even when only one name wraps.

Untick it if you would rather keep the header one line tall and let long names trim.

### Divider lines

**Draw divider lines at band boundaries** puts a vertical rule wherever one band ends and the next
begins, running from the header through every body row. Weight and colour are adjustable. The
dividers follow the band grouping even with the band row itself hidden, so you can use bands purely
as a way to define column groups if you don't want the extra header tier.

For a rule that doesn't line up with a band, tick **Div** beside any column in the list on
**⚙ → Table → Columns**. That draws a divider immediately to the left of that column, in the same
weight and colour. The two sources add together: band boundaries give the automatic set, ticked
columns add to it, and a column that is already a band boundary is not drawn twice. Turn
**Draw divider lines at band boundaries** off to leave only the columns you ticked — that's how to
place rules by hand with no bands involved at all. Both kinds carry into the PDF.

In the PDF each divider is drawn **once per page** as a single filled rectangle running the full
height of the table, from the top of the header block to the bottom of the last row. It used to be
drawn cell by cell, which meant it was interrupted wherever a row spanned the full width — every
group band punched a hole in every divider, and a divider that fell inside a merged header band left
a second hole in the header. A rule with holes in it reads as fading in and out, which is exactly
what it looks like.

The rectangle is a whole number of points wide and is centred exactly on the column boundary it
belongs to. An earlier build also rounded the left edge to a whole point, on the theory that
identical geometry would snap to the device pixel grid identically; it did, but it also let a rule
sit up to half a point off its own boundary, so it no longer lined up with the header separator and
the band edge above it and the whole rule read as slightly out of true. Exact centring costs a
one device-pixel difference in rasterised width between one rule and the next at low zoom, which is
the physical limit for a vector rule at a fractional position — no rule is ever a fraction of its
neighbour's weight.

## Column widths and row heights

**⚙ → Table → Columns** lists every column with a width box in pixels; leave one blank for
automatic. A column with a set width always wraps its text inside that width — that's the point of
setting one — and the row grows as tall as the tallest wrapped cell. No manual row-height setting is
needed; that's ordinary table behaviour once the text can wrap.

**Wrap long text in every column** applies the same wrapping to unsized columns.
**Max wrapped lines** caps how far a row can grow: `0` means no limit, any other value trims the
cell at that many lines (the full text stays available on hover). Widths carry into the PDF,
converted from pixels to points.

## Font sizes and alignment

The column list on **⚙ → Table** carries three per-column settings: **Width**, **Size** (font size in
pixels) and **Align**. Align overrides the default, which right-aligns numbers and left-aligns
everything else — useful for a numeric code that reads better on the left, or a short flag centred
in its column.

**⚙ → Type** holds the global sizes: body, column header, merged band and group band, each
independent.

**Alignment** covers both axes. **Column headers** defaults to centred, and can instead match each
column's own alignment or force left/right regardless of what the data does. **Header vertical** and
**Merged band vertical** put the text at the middle (default), top or bottom of its cell — worth
setting when a two-line date label or a tall band leaves the single-line headers beside it looking
stranded.

## Empty values

**⚙ → Table → Empty values** controls what a null looks like: **Blank**, **0**, an en dash, or
custom text such as `n/a`. Tableau reports nulls as a null value, an empty formatted value, or the
literal `%null%`, or the word `Null` depending on the field type and data source — all of them are
treated the same way.

Choosing **0** also makes conditional-formatting rules read those cells as zero rather than skipping
them, so an "is negative" or `≤ 0` rule behaves as you'd expect.

## Sort order

**⚙ → Table → Sort order** is an ordered list of levels; each one breaks ties in the level above.
The default is a single level — **Group By value, ascending** — which is what keeps each group's
rows together in one band. Add, reorder and remove levels with the arrows.

Clicking a column header sorts by that column, and because **Keep the Group By level first** is on,
it sorts *within* each group rather than scattering the bands. Shift-click a second header to nest
another level beneath it. A third click on a header drops that level again.

Sorted columns are not marked in the header by default. Tick **Mark sorted columns with an arrow**
to show a caret instead, numbered (`▲2`, `▼3`) once more than one level is active.

Turning **Keep the Group By level first** off makes a header click the top-level sort, which
reorders rows across the whole table — expect the group bands to fragment into many small bands,
since a band is drawn wherever the group value changes.

Ties fall back to Tableau's own row order, so the sort is stable: rows that match on every level
stay in the order the worksheet produced them.

## Conditional formatting

The **Rules** tab formats cell values. Each rule picks a target — one column, all numeric columns,
or every column — a condition, and what to apply: font weight (bold / regular / leave alone),
italic, text colour, fill colour. **+ Negatives in red** adds the most common rule in one click.

Conditions: is negative / positive / zero / not zero, `>`, `≥`, `<`, `≤`, `=`, `≠`, between,
outside, text contains, text is, is blank, is not blank.

**Compare to value** tests against a number or date you type. **Compare to field** tests against
another column in the same row — that's how you flag a date that has slipped past its scheduled
counterpart (`CN Comp Date ≥ POO_SCH_Start_DATE`). Date columns compare chronologically, so a typed
`6/1/2026` is read as a date rather than as the number 6.

Rules run top to bottom and **stack**, so a later rule can override an earlier one's weight or
colour. A common setup is one broad rule (all numeric columns, negative → bold red) followed by
narrower exceptions (`FQI` at least 80 → bold green on a light green fill).

Numbers are read from the underlying value first, falling back to the displayed text with currency
symbols, thousands separators, percent signs, and accounting-style parentheses handled — so
`(1,250)` matches "is negative" and `83%` matches "is at least 80".

Formatting carries into the PDF, including bold, italic, text colour and cell fill. CSV is values
only, as it has no formatting to carry.

This is the closest replacement for the paired `(B)` / `(R)` fields the current crosstab uses to
fake bold and red text: drop only the plain version of each field and express the emphasis as a
rule instead.

## Date-labelled week headers

Turn on **⚙ → Headers → Label week columns with their date range** and name the parameter that holds
the report's through-date — `Enter Through Week Ended Date` in this workbook. The extension reads it
with `findParameterAsync`, computes each column's range, and subscribes to `ParameterChanged`, so
moving the through-date re-labels every week column immediately. No formulas, no retyping.

**Apply to** picks how columns are chosen:

- **Columns matching a name prefix** — any header starting with the prefix (default `Week`) and
  ending in a number. The *number* decides which period it is, not the column's position, so
  reordering can't scramble the dates. Good when the columns are already named `Week 1`…`Week 7`.
- **Columns I pick below** — a checklist of the sheet's actual columns, each with its own period
  number and a live preview of the range it resolves to. Use this to label measures that aren't
  named after a week (`POO`, `FS Sch Start`, a bare `Total Change`), or to label only some of the
  week columns. **Auto-number from names** pre-fills it from any headers ending in a digit, so you
  can start from the prefix behaviour and adjust.

The dialog gets the column list through `displayDialogAsync`'s payload — a settings dialog has no
worksheet access of its own — and stores your picks keyed by *field* name, so header overrides and
column reordering don't break the mapping.

Period numbers follow the same direction rule, and they aren't limited to 1 and up: **0** is one
period later than period 1 and negatives keep going forward, which is how you label a "next week"
column. `Days per column` (default 7) handles bi-weekly or 10-day buckets.

`Week 1 is` sets the direction. This workbook is configured for **the week ending on the
through-date**, so with a through-date of 8/7/26:

| Column | Range |
|---|---|
| Week 1 | Aug 1–7 |
| Week 2 | Jul 25–31 |
| Week 3 | Jul 18–24 |
| … | … |

The other two options are *the oldest week* (weeks run forward, the last column ends on the
through-date) and *the first week after* (a forward-looking schedule).

By default a labelled column is **retitled by its period**: the first line becomes `Week 3` and the
date range sits underneath, so a column actually named `CN Comp Days` reads as a week rather than as
a field name. The wording comes from **Period label** (default `Week {n}`, where `{n}` is the period
number) — set it to `Wk {n}`, `Period {n}`, or just `Week` and the number gets appended.

Untick **Include the period number** to drop the number altogether: every period column then reads
just `Week`, with the date range underneath telling them apart. Untick **Rename the labelled columns
by period** instead to keep the original field names as the first line.

Anything typed into **Header overrides** always wins over the period label, so you can still force a
specific column's wording.

Labels render as a lighter second line under the header, or replace it entirely — your choice
under `Show as`. Five formats: `Aug 1–7` (collapses the month when both ends share it),
`Aug 1 – Aug 7`, `8/1–8/7`, `08/01 – 08/07`, `08/01/26 – 08/07/26`. The picker previews the
resolved dates in whichever format you pick. Hovering a dated header shows the full range and
where the date came from. The CSV export carries the range in its header row too.

A **fallback date** can be typed in for cases where the parameter isn't found — a sheet that isn't
wired to it yet, or previewing in a browser. The parameter always wins when it resolves.


### Gridline width

**⚙ → Colors → Gridline width** sets the thickness of the rules between rows and columns, in pixels,
from 0 to 4 in half-pixel steps. It is separate from **Gridlines** directly above it, which sets
their colour, and separate again from divider weight — dividers are the heavy vertical rules at band
boundaries, gridlines are the light grid the whole table sits on.

Set it to **0** for no gridlines at all, which is the cleanest look when row shading is doing the
work of separating rows. On screen the line under the column headers is drawn at twice whatever you
pick, which is the proportion the table has always had, so the header stays visibly separated at any
setting.

The PDF uses the same value, converted at the usual 0.75pt per pixel — so 1px on screen prints as a
0.75pt rule, and 0 prints with no cell boxes at all. The grid used to be hardcoded at 0.4pt in the
PDF regardless of anything, which is why printed output looked lighter than the screen. The one
difference from the screen is the header underline: the PDF strokes each header cell as a box at a
single weight rather than doubling its bottom edge, because a doubled box would thicken the vertical
separators inside the header block too.

## PDF export

The **⤓ PDF** button renders every filtered row — not just the page on screen — across as many
pages as it takes:

- Column headers repeat on every page, including the two-line date labels and the merged bands —
  a column with no band over it takes the full height of both tiers here exactly as it does on
  screen, rather than carrying a strip of empty grey.
- Group bands are preserved, and a page that carries a group over from the last one repeats its
  band with a *"continued"* marker, the same as the view.
- The subtitle line carries the through-date, the active quick-filter text, the row count, and your
  own note, so a printout is self-describing. The row count can be switched off, and the noun is
  yours to set — a schedule counts *homes*, a punch list counts *items*. Give the plural and the
  singular (**⚙ → PDF → Call them**) and the export picks the right one: *368 homes*, *1 home*.
- Footer has `Page X of Y` and a generated-on timestamp.
- **Rows are never split across a page break.** A row whose wrapped text won't fit in what's left of
  the page moves down whole and starts at the top of the next one, so you never get an orphaned
  half-row sitting under the repeated header.
- **Legal landscape by default** — 1008 × 612 pt, about 27% more width than Letter landscape, which
  is what a 20-plus column report needs. Letter, Tabloid, A4 and A3 are also available, in either
  orientation.

Filename is the title slugged plus the date, e.g. `Production-Trends-Overview-2026-08-12.pdf`.

### Rows per page

**⚙ → PDF → Rows per page** decides how the export is broken up.

**Match the table's rows per page** (the default) makes a printed page carry the rows one on-screen
page carries — set the table to 30 and page 4 of the PDF holds the same thirty rows as page 4 of the
view, so the two can be read side by side. Each page is a separate layout pass, but the column widths
are measured once across the whole report and pinned, so every sheet shares one grid rather than
drifting.

**The count is fixed.** Set 15 and every page carries 15; only the last page carries the remainder,
however small that is. Nothing is capped to what happens to fit, averaged across pages, or evened out
at the end — a run of pages with different counts is much harder to read against the view than a short
final page is.

Two things are done to make the count achievable rather than to compromise it:

- The header block is measured from where the first row actually landed, not by adding up the header
  rows. A heading that spans both tiers reports its whole two-row height in the first of them, so
  trusting that sum made the height budget a full row too small.
- The **repeated group band** at the top of a carry-over page is dropped on any page where keeping it
  would push the rows past the sheet. The band is a convenience, not data — the *"continued"* marker
  still appears in the header area — and giving it up buys back exactly the row's worth of height that
  usually decides whether the count fits. The toast reports how many pages this affected.

If the rows still do not fit, those pages run over onto the next sheet rather than the count being
quietly reduced, and the toast tells you what to do about it: *"15 rows do not fit the sheet on 5
pages, so those run over. About 14 fit at this font size and row padding."* Take that number, or trim
the font size or row padding, and the pages come out uniform.

The export deliberately does **not** shrink the type to force more rows on. An earlier build did, and
it made every size control feel broken: raise the body size and the vertical fit scaled it straight
back down, so nothing changed; raise the row padding and the type shrank to pay for it, leaving the
rows the same height in smaller print. Your sizes are printed as set.

**As many as the page holds** ignores the table's page size and packs each sheet full — fewer pages,
less paper, but the PDF no longer lines up with the view. It leaves the pagination to the renderer, so
its final page carries whatever remains, as in any paginated document.

### Header image

**⚙ → PDF → Header image** puts a logo or banner above the title on the PDF. Pick a file, set its
height in points, and choose left, centred or right; the width follows from the image's own aspect
ratio. It can print on every page or on the first only.

The picture is stored inside the workbook, so it travels with the published view and needs no
hosting — but that also means it has to stay small. The dialog redraws whatever you pick at print
resolution before encoding it, which turns a multi-megabyte logo into a few tens of KB, and it shows
you the stored size and the printed width so there are no surprises. Anything that still encodes
above 600 KB is refused rather than quietly bloating the workbook.

### Fitting to one page wide

**Always fit the table to one page wide** (on by default, on the PDF tab) guarantees the table never
runs off the right edge of the paper, however many columns you have.

It works by measuring rather than guessing. The table is laid out once into a throwaway document and
the width it actually used is read back. If that fits, nothing changes. If it overflows — which
happens when the column widths you set add up to more than the page — every font size, every pad and
every column width is multiplied by exactly the shortfall and the table is laid out again. Type
metrics scale linearly, so a single correction lands it inside the margin rather than creeping up on
it. The scale that was applied is reported in the toast, e.g. *"2 pages, 92 rows exported. Scaled to
71% to fit the page width."*

The measurement pass costs a second layout, so it is skipped whenever overflow is arithmetically
impossible — the floor is your explicit widths plus 10pt per auto-width column, and if that already
fits the page the export goes straight through in one pass. In practice you only pay for it when you
have set wide fixed widths.

If a scale comes back uncomfortably small, the fix is not to turn the option off — it is to widen the
paper (Tabloid) or trim the column widths that are forcing it. Turning it off just puts the overflow
back.

### What carries over from the other tabs

The PDF is not a separate style — nearly every formatting choice on the other tabs is reproduced:

| Setting | In the PDF |
| --- | --- |
| Per-column **Width** | Converted px → pt at 0.75 |
| Per-column **Align** | Applied to the cells, overriding the numbers-right default |
| Per-column **Size** | Kept as a ratio (see below) |
| **Header alignment**, including *match the column* | Applied to the header row |
| **Header vertical** / **Merged band vertical** | Applied as autoTable `valign` |
| **Merged band alignment** (left / centred / right) | Applied to the band row |
| **Merged band font size**, **group band font size** | Kept as a ratio |
| **Row padding** | Halved into points on data rows, so the default 4px still gives the old 2pt; header rows get the full value, because a band name wrapped onto three lines needs more air above it than a line of figures does |
| **Alternating row shading**, **row numbers** | On or off, as set |
| **Wrap long text in every column** | Off, an unsized column is cut with an ellipsis, same as on screen |
| **Max wrapped lines** | Text is cut to that many lines *before* layout, so the row is genuinely shorter |
| **Body / header weight** | Bold when set to 600 or heavier |
| **Font family** | Mapped onto the nearest core PDF font — Helvetica, Times or Courier |
| **Conditional formatting rules** | Bold, italic, text colour and cell fill, per rule |
| **Divider lines** — band boundaries and per-column | Drawn down the full column, in your weight and colour |
| **Header text overrides and hidden headers** | Renamed and blanked the same way |
| **Date-labelled headers** and band date ranges | Included, stacked onto a second line |
| **Null display** | Blank, `0`, dash or custom text, as chosen |
| Colours — header, band, group, border, text, row banding | All applied |
| **Gridline width** | Same weight as the screen, converted at 0.75pt per pixel; 0 prints no grid |

One thing deliberately doesn't carry: **minimum column width**. The screen scrolls sideways and a
page doesn't, so a 70px floor across twenty-odd columns would push the table off the paper. Set
explicit widths instead.

### Font sizes

**Use the table's font sizes** on the PDF tab (on by default) makes the **Type** tab drive both the
view and the PDF — pixels become points at 0.75, the same ratio used for column widths, so 12px on
screen prints at 9pt and changing one changes both. The two point sliders on the PDF tab are greyed
out while this is on, because they aren't being read.

Untick it when the printout wants a size of its own — twenty-odd columns often want 7pt on paper
even at 12px on screen. The Type tab then affects the view alone and the two sliders take over.

Whichever mode you are in, the size you set is the size that prints. Nothing scales it back down to
make rows fit a page; if they don't fit, the page spills (see **Rows per page** above). The only
thing that still scales type is the width fit, and only when the table is wider than the paper —
which the toast reports as a percentage.

Either way, a per-column, band or group size is carried as a *ratio* against its on-screen base, so a
column set to 20px against a 12px body comes out 1.67× the PDF body size. Relative emphasis survives
in both modes; with matching on, the absolute size does too. And in both modes the fit settings shrink
everything proportionally if the result won't fit the paper — so raising the size may show up as a
lower fit percentage in the toast rather than bigger type, if the table was already at the limit.

A note on the two wrapping settings, because they change what a printed report contains. **Wrap long
text in every column** off means an auto-width column that doesn't fit is cut with `...` on paper
exactly as it is on screen — faithful, but text is lost from the page. Tick it and those columns wrap
instead, which keeps every character at the cost of taller rows and more pages. A column with an
explicit width always wraps in both places, since that is the point of setting a width. **Max wrapped
lines** then caps how far any row can grow: the text is trimmed to that many lines against the
column's measured width *before* the table is laid out, so the row really is shorter rather than
being tall and half empty.

Whichever way those are set, a row is kept whole across page breaks — it moves to the next page
rather than being cut in half by one. The only case that can't be honoured is a single row taller
than the whole printable area; cap it with **Max wrapped lines** if you ever hit that.

## Settings reference

Every setting is stored in the workbook via `tableau.extensions.settings`, so it travels with the
published workbook and applies for viewers.

**Pagination** — rows per page, size choices, control bar position (top/bottom), numbered page
buttons and how many to show, go-to box, rows-per-page dropdown, row counter, quick filter,
CSV button.

**Table** — group bands on/off, repeat band across page breaks, group row counts, band label
prefix, per-column widths, font sizes and alignment, freeze first N columns, minimum column width, row padding, row banding,
row numbers, wrapping and max wrapped lines, null display, the nested sort-level list and whether
the Group By level stays first, header-click sorting, select-mark-on-row-click.

**Headers** — per-column header text and hide ticks; merged bands on/off, the per-column band picker,
band-only-where-named with top-aligned spanning headings, band wrapping,
overall-date-range on/off with stacked or bracketed placement, band alignment, and divider lines
with weight and colour; the per-column **All** tick that shows the aggregate span under one header; date labels on/off, parameter name, fallback date,
target mode (name prefix or a per-column picker), week direction, days per column, period-label
template with an optional period number, second-line vs. replace, and date format.

**PDF** — show the button, title, note, row count on/off with its own singular and plural noun,
repeat title block, page numbers, timestamp, group bands, header image (file, height, position,
repeat), orientation, paper size (Legal landscape by default), rows per page mode, use the table's
font sizes or the PDF's own body and header size, margin, fit the table to one page wide.

**Type** — font family (locally installed fonts only; Desktop blocks web-font downloads), body,
header, merged band and group band sizes, body and header weight, and horizontal plus vertical
alignment for headers and merged bands.

**Rules** — conditional formatting: target, condition, threshold values, font weight, italic, text
colour, fill colour, per rule.

**Colors** — five presets (Light, Slate, Forest, Mono print, Dark) plus individual pickers for
background, body text, row shading, gridlines and their width, hover, header background/text, merged band
background/text, group band background/text, and the accent used for the active page button and
sort carets.

## Troubleshooting

**Broken-page icon, empty grey view.** Tableau parsed the manifest (the Columns / Group By shelves
appear on the Marks card) but could not fetch the HTML. Either the local server isn't running, or
the URL in `paginated-table.trex` doesn't match where the files actually are — a repo renamed, Pages
not finished building, or the repo still private. Open the URL from the manifest in a browser;
whatever the browser does, Tableau will do.

**Yellow "demo data — not connected to Tableau" chip.** The page rendered but found no Tableau host.
Hover the chip for the reason. Inside Tableau this should never appear; if it does, the API `.js`
file is missing from the folder next to `index.html`.

**"Add fields to build the table".** The extension is connected and working — it just has no
columns yet. Pills on **Detail** don't become columns; drag them onto the **Columns** shelf.

**A field won't drop onto the Columns shelf.** Re-load the `.trex` — manifest changes only take
effect when the extension is re-added. Builds before 12 Aug 2026 restricted the shelf to string and
numeric fields via a `<data-spec>` block, which silently rejected dates; that restriction is gone,
and the shelf now takes any field type. The only remaining limit is 30 fields.

**Rows look broken or misaligned after setting a column width.** Fixed on 13 Aug 2026. Earlier
builds applied the line clamp to the table cell itself, which took it out of the table layout and
let sized columns drift out of their rows. Re-copy `index.html`.

**Nulls still show as "Null" or "%null%".** Also fixed on 13 Aug 2026 — Tableau writes nulls three
different ways depending on the field type, and only one of them was being detected. Re-copy
`index.html` and pick what you want under ⚙ → Table → Empty values.

**"PDF libraries missing".** `jspdf.umd.min.js` and `jspdf.plugin.autotable.min.js` aren't next to
`index.html` at whatever URL the manifest points to. Everything except PDF export still works.

**A picked column shows no date.** In *Columns I pick below* mode a column is labelled only when
it's ticked *and* has a period number. If the picker shows "No column list available", the dialog
was opened without a column payload — close it, make sure the sheet has fields on the Columns
shelf, and open ⚙ again.

**Week headers show no dates.** The parameter name in ⚙ → Headers has to match exactly, and the
sheet has to be able to see that parameter. Hover a week header: if the tooltip shows only the field
name, the anchor date never resolved. Type a fallback date to confirm the rest of the feature works,
then fix the name.

**"Tableau did not finish the extension handshake within 15 seconds."** On Server or Cloud, the URL
isn't allowlisted (or is allowlisted without full data access). On Desktop, this usually means the
page was served but its scripts weren't.

## Known limitations

- **Sorting and filtering are client-side**, over the summary data Tableau hands the extension.
  Very large marks counts (tens of thousands of rows) will feel heavy; filter at the data source
  first.
- The extension replaces the entire worksheet viz, so Tableau's own row/column headers, totals,
  and tooltips do not appear. Hovering a truncated cell shows its full text as a title attribute.
- `selectMarksByValueAsync` (row click → select in Tableau) matches on the group field plus the
  first two non-numeric columns. If those don't uniquely identify a mark, the selection is
  approximate; the option is off by default.
