---
name: highcharts-orbit-design
description: "Design profiles for Highcharts Orbit demos: the shared page template contract (CSS custom properties, structure, industry icon chip, the collapsed-by-default how-to slider and its screenshot pipeline, header controls), the four named skins with exact token values (highcharts, editorial, console, aurora), runtime skin switching without chart recreation, and how Orbit chrome follows the page color-scheme in dark mode. Use when styling a new Orbit demo, switching a demo to a different design, adding a skin, building the guide box screenshots, or creating a new design profile. Theming API mechanics live in highcharts-orbit-facts; hosting in highcharts-orbit-hosting."
---

# Orbit demo design profiles

How Orbit demos get their look. A demo changes design by swapping a skin,
never by rewriting CSS. Theming API mechanics live in
`highcharts-orbit-facts` section 5; this skill holds the template contract,
the concrete skins and the runtime switching pattern.

## 1. Template contract

Required custom properties, defined on `:root` (light) and
`:root[data-theme="dark"]` (dark), per skin under `:root[data-skin="NAME"]`:

```
--page-bg      page background
--card-bg      panel, KPI card and header background
--text         primary text
--muted        secondary text
--border       1px borders
--accent       wordmark span, journey tool names, hover accents
--shadow       card shadow
--demo-accent  per-demo accent for the industry icon chip (section 1b)
```

Also stamp the native colour scheme so browser-drawn UI (scrollbars, form
controls, popups) **and Orbit's own chrome** match:

```css
:root { color-scheme: light; }
:root[data-theme="dark"] { color-scheme: dark; }
```

Page structure every skin styles: sticky `header` (wordmark link, industry
chip, demo tag, controls), `main` with h1 trigger question, `.setup`
paragraph, the `.panel--guide` how-to slider, `.kpis` grid of `.kpi` cards
with `data-orbit-context`, `.panel` chart blocks (some inside `.grid2`
two-column rows), `#orbit-note` fallback, `footer`.

Layout constants: `main` max-width 1300px, padding 32px 20px 64px; gap 18px
(24px inside a slide); panel padding 24px; KPI padding 18px 20px; card radius
16px (6px editorial, 4px console, 22px aurora); chart height 380px (340px
short, 300 to 340px gauge); h1 2.625rem/1.15; body line-height 1.3. `.grid2`
is `grid-template-columns:1fr 1fr`, collapsing under 900px.

Header markup: the wordmark is a link back to the demo index
(`<a class="wordmark" href="/">Highcharts <span>Orbit</span> demos</a>`, or
the absolute deploy URL when the demos are also embedded elsewhere), then the
industry chip, the demo tag, then `.controls` holding the skin picker and the
icon-only theme toggle.

## 1b. Industry icon chip

One inline SVG industry icon per demo in a 40px rounded chip, in the header
and on each index card. Inline SVG only (24x24 viewBox,
`stroke="currentColor"`, stroke-width 1.8, round caps, `fill="none"`).

```css
.industry-chip {
  width:40px; height:40px; border-radius:10px; flex:none;
  display:flex; align-items:center; justify-content:center;
  background:color-mix(in srgb, var(--demo-accent) 14%, transparent);
  color:color-mix(in srgb, var(--demo-accent) 62%, var(--n900));
}
:root[data-theme="dark"] .industry-chip { color:color-mix(in srgb, var(--demo-accent) 75%, #fff); }
```

Chip is `aria-hidden="true"`. Index cards get
`transition: transform .15s ease` and `:hover { translateY(-2px) }`.

| Demo | Industry | Icon motif | --demo-accent |
|---|---|---|---|
| 01 | Internal dashboards / logistics | package | #2caffe |
| 02 | Embedded / SaaS e-commerce | shopping cart | #00e272 |
| 03 | Financial services | trending-up | #544fc5 |
| 04 | Scientific / industrial | pulse line | #fe6a35 |
| 05 | Publishers / media | newspaper | #d568fb |
| 06 | Enterprise software | server stack | #2ee0ca |

## 1c. How-to slider (the guide box)

The investigation list is a step slider titled **How to use Orbit on this
demo** in a `.panel--guide` panel, tinted clearly enough to read as
instruction rather than data:

```css
.panel--guide {
  background:color-mix(in srgb, var(--accent) 11%, var(--card-bg));
  border-color:color-mix(in srgb, var(--accent) 42%, var(--border));
}
:root[data-theme="dark"] .panel--guide {
  background:color-mix(in srgb, var(--accent) 17%, var(--card-bg));
}
```

A skin with a saturated accent needs a lighter hand than 11/17 or the box
turns into a colour block; console overrides it to 8/9.

Each slide is one Orbit feature: "Step X of Y" micro-label, the tool name in
`--accent`, what it shows in this demo, and a **How:** line naming the exact
menu path (check it against `highcharts-orbit-facts` section 5a; a wrong path
is the easiest error to ship here). Slide layout is
`grid-template-columns:minmax(0,1fr) minmax(0,1fr)`, copy left, screenshot
right, `align-items:start` so both columns start at the top, and
`.journey-copy { text-align:left; }`. Right-aligned copy was tried on
2026-08-31 and rejected: a ragged left edge makes an instruction paragraph
harder to read. Top aligned yes, right aligned no.
A slide with no image gets `.journey-slide--nomedia` and goes single column,
but prefer giving every slide a shot; under 980px everything stacks.

**Fixed height, closed by default.** The slider is a fixed
`height:230px` (`padding:14px 46px 0`) so the box does not jump between
slides of different text length. Size it from the tallest slide, not by
guessing: at the 1300px max width the media column is about 190px tall
(3:1 of a 568px column), so 14 top + 190 + 10 + the 8px dots row is roughly
215, and 230 leaves a small, deliberate margin. Re-measure after any layout
change rather than leaving a comfortable number in place, because dead space
under the slide is the first thing people notice.

The box **ships collapsed**: the markup carries
`class="panel panel--guide is-collapsed"` so there is no open-then-close
flash, and the script opens it only when `localStorage` holds
`orbit-demo-guide = 'open'`. Any other value, or none, stays closed. The
collapsed panel gets `padding:18px 24px` so it reads as a slim bar.

The header is a `.guide-head` flex row and the whole row is clickable:
a 34px accent-tinted `.guide-badge` (inline book icon), the `h2` at
1.375rem, a `.guide-count` that the script fills with the slide count
("6 steps"), and a pill `button#guide-toggle` on the right whose label
switches between **Show** and **Hide** next to a chevron that rotates
`-90deg` when collapsed. The row tints on hover so the whole bar reads as a
control, not a heading. `aria-expanded` and `aria-label` follow the state.

**Media sizing.** Every screenshot is normalised to one fixed canvas,
**1200x400 PNG**, white ground. The figure is
`aspect-ratio:3/1; justify-self:end; width:100%; display:flex;
align-items:flex-start; justify-content:flex-end`, and the image is
`width:100%; height:100%; object-fit:contain; object-position:right center`.
Same size on every slide, top aligned, flush right. PNG, not JPG: these are
UI screenshots, and JPG smears the small type. Images load eagerly (the box
is above the fold).

**Navigation.** Prev and next are full-height edge rails, not inline
buttons: the slider is `position:relative`, and each button is
`position:absolute; top:14px; bottom:40px; width:36px`, left or right (the
bottom inset clears the dots row), holding a chevron SVG, tinted
`color-mix(in srgb, var(--accent) 7%, transparent)` with an accent-mixed
border and a stronger hover. The dots row sits centred below the slide; the
active dot uses `var(--demo-accent)`; the slider loops.

### Screenshot pipeline (shots/)

`shots/base-toolbar.png` is the one real capture: the demo's Orbit toolbar in
light mode at 2x (2420x76), taken with html2canvas. Every menu shot is that
capture with the menu composited in with PIL, because html2canvas does not
paint Orbit's portal menus.

Rules that make the shots usable:

1. **Menu content is read from the live DOM first**, never remembered. Labels
   and one-line descriptions must match Orbit exactly (facts skill, 5a). A
   drawn menu with invented items is worse than no shot. The same goes for
   panels: draw only rows that were read from the open drawer.
2. Draw at 2x and downscale with LANCZOS. Card `#ffffff`, ink `#141418`,
   muted `#6e6e76`, border `#e2e2e7`, highlight row `#f3f3f5`, radius 9px,
   rows 44px without a description and 56px with.
3. Position the menu under its real trigger using the offsets in the facts
   skill, so the shot reads as the actual toolbar.
4. **Kill whitespace by cropping, not by padding.** Crop the board to the
   content bounding box, then crop its width to
   `max(trigger right edge, menu right edge, content height x 3)` before
   fitting into 1200x400. Without that step short menus land in a sea of
   white, which was the single loudest complaint about the first set.
5. One shot per distinct menu path, named for the path
   (`menu-analyze-trends.png`, `menu-analyze-indicators.png`,
   `menu-view-distribution.png`, `topbar-compare.png`). A slide must use the
   shot whose highlighted row is the tool the slide teaches. Two slides about
   different tools must not share one shot.
6. html2canvas **hangs on `color-mix()`** ("unsupported color function"):
   inject an override stylesheet with flat colours before capture, and
   capture the toolbar alone rather than a whole panel at 2x.
7. Delete shots nothing references. An unused file in `shots/` means either a
   slide lost its image or a shot was built for a path no demo teaches.
8. The shots are captured in light mode and keep a white ground in every
   skin, which is correct: they are pictures of Orbit's UI, not page
   furniture. Give the figure a border and the card shadow so it sits as a
   framed screenshot rather than a hole in a dark page.

## 1d. Header controls

The theme toggle is icon-only: a moon in light mode, a sun in dark mode (it
shows what you switch TO), with `aria-label` and `title` updated on every
paint.

The skin picker is **an own dropdown, not a native `<select>`**: a browser's
native popup list is drawn outside the page's styling and does not reliably
follow the tokens in dark mode (verified: setting `color-scheme` and option
colours still leaves it browser-dependent). Markup is a
`button#skin-btn` (`aria-haspopup="listbox"`, `aria-expanded`) plus a
`ul#skin-menu[role=listbox][hidden]` of `li[role=option][data-skin-value]`,
absolutely positioned under the button, styled with `--card-bg`, `--border`,
`--shadow`, accent-tinted hover, and the selected item in `--accent`. It
closes on outside click and on Escape, and items respond to Enter and Space.

```css
.controls { margin-left:auto; display:flex; align-items:center; gap:10px; }
.skin-picker { position:relative; }
.skin-btn { font:inherit; font-size:.875rem; color:var(--text); background:var(--card-bg);
  border:1px solid var(--n200); border-radius:8px; padding:7px 10px;
  display:flex; align-items:center; gap:6px; cursor:pointer; }
.skin-menu { position:absolute; right:0; top:calc(100% + 6px); z-index:20; margin:0; padding:4px;
  list-style:none; min-width:100%; white-space:nowrap; background:var(--card-bg);
  border:1px solid var(--border); border-radius:8px; box-shadow:var(--shadow); }
#theme-toggle { color:var(--text); background:transparent; border:1px solid var(--n200);
  border-radius:8px; padding:7px; display:flex; align-items:center; justify-content:center; cursor:pointer; }
```

## 1e. Runtime skin switching (verified live 2026-08-30)

Skins switch with **no chart recreation, so Orbit stays attached**: verified
identical toolbar and chart counts before and after, with series colours
changing. Pattern:

```js
Highcharts.charts.filter(Boolean).forEach(function (c) {
  c.update({ palette: skin.palette, chart: { style: { fontFamily: skin.font } } }, false);
  c.redraw(false);
});
document.documentElement.setAttribute('data-skin', name);
try { localStorage.setItem('orbit-demo-skin', name); } catch (e) {}
```

`palette.colorScheme: 'light dark'` makes chart colours resolve through CSS
`light-dark()`, which follows the **system** preference unless a
`.highcharts-light` / `.highcharts-dark` class sits on a parent. The theme
toggle sets that class on `<html>` along with `data-theme`. Consequence when
testing: setting `data-theme` by hand in the console leaves the class stale
and the charts render in the wrong theme while the page looks right. Always
drive the real toggle button, or set both.

Page tokens come from CSS (`:root[data-skin="NAME"]`), chart colours from
`palette` in the same skin object. The inline head script reads the stored
skin and stamps `data-skin` before paint, and the main script feeds the
active skin's font and palette into `Highcharts.setOptions` before the charts
are created, so there is no flash and no first-render mismatch. Note that
`palette` injects `--highcharts-*` CSS variables with higher specificity than
`:root`, so overriding those variables by hand does not work; go through
`update()`.

## 2. Skin: highcharts.com (default, verified)

Read from www.highcharts.com live 2026-08-29, verified in both themes.

Font: `'IBM Plex Sans', sans-serif` (Google Fonts, 400/500/600/700).

```
--n0:#fff; --n25:#f7f7f8; --n50:#f2f1f4; --n100:#e3e3e8; --n200:#c8c7d1;
--n300:#acabba; --n400:#918fa3; --n500:#75738c; --n600:#5e5c70; --n700:#474554;
--n800:#2f2e38; --n850:#23222a; --n900:#18171c; --n950:#0a090d;
--brand-400:#a5acff; --brand-500:#8791ff; --brand-600:#626bd0; --brand-700:#4a55c9;
```

Light: page `--n50`, card `--n0`, text `--n900`, muted `--n600`, border
`--n100`, accent `--brand-600`, shadow
`0 1px 2px rgba(0,0,0,.06), 0 3px 3px rgba(0,0,0,.05)`.
Dark: page `--n950`, card `--n900`, text `--n25`, muted `--n300`, border
`--n800`, accent `--brand-400`, shadow
`0 1px 2px rgba(0,0,0,.4), 0 3px 3px rgba(0,0,0,.3)`.

Semantic: bad value `#dc2626` light / `#f87171` dark; warning note border
`#d97706` text `#b45309` light, text `#fcd34d` border `#92400e` dark; gauge
bands ok `#00e272`, warn `#f9c80e`, alert `#fa4b42`. These are shared across
skins, so check them against any new accent.

```js
palette: {
  colorScheme: 'light dark',
  light: { backgroundColor: '#ffffff', neutralColor: '#18171c', highlightColor: '#626bd0' },
  dark:  { backgroundColor: '#18171c', neutralColor: '#f7f7f8', highlightColor: '#8791ff' },
  colors: ['#2caffe', '#544fc5', '#00e272', '#fe6a35', '#6b8abc',
           '#d568fb', '#2ee0ca', '#fa4b42', '#f9c80e']
}
```

## 3. Skin: editorial (built 2026-08-30, verified live)

An original palette and type pairing in the editorial data-report genre:
warm paper ground, near-black warm ink, serif display over a grotesk body,
quiet borders, small radii, muted earth series colours. Values are original;
nothing is copied from another site.

Fonts: **`'Newsreader'`** (display: h1 weight 500, KPI values 600, tool names
and card headings 600, letter-spacing -.005em, -.015em on h1) and
`'Instrument Sans'` (body), both Google Fonts.

Display-face note, learned 2026-08-30: a high-contrast display serif with
hairline strokes (Instrument Serif was tried first) is not legible enough at
KPI and label sizes on a warm ground. Newsreader is drawn for screen reading,
has moderate stroke contrast and open counters, and keeps the editorial
character. Any replacement display face must be checked at the smallest size
it is used at (tool names, roughly 20px), not just in the h1.

```
:root[data-skin="editorial"] {
  --page-bg:#f4f1ea; --card-bg:#fffdf8; --text:#1c1a17; --muted:#6b6459;
  --border:#e2dbcd; --accent:#10473f;
  --shadow:0 1px 2px rgba(28,26,23,.05);
}
:root[data-skin="editorial"][data-theme="dark"] {
  --page-bg:#14130f; --card-bg:#1e1c17; --text:#f2efe6; --muted:#a49b8b;
  --border:#33302a; --accent:#7fc0a9;
  --shadow:0 1px 2px rgba(0,0,0,.5);
}
```

Radii: cards and panels 6px, controls and buttons 4px. Micro-labels
(`.kpi .label`, `.journey-step`) get `letter-spacing:.08em`.

```js
palette: {
  colorScheme: 'light dark',
  light: { backgroundColor: '#fffdf8', neutralColor: '#1c1a17', highlightColor: '#10473f' },
  dark:  { backgroundColor: '#1e1c17', neutralColor: '#f2efe6', highlightColor: '#7fc0a9' },
  colors: ['#2e6f5e', '#c96f3f', '#4a6fa5', '#b8894a', '#7a5c8e',
           '#4f8f8a', '#a8544e', '#6b8f3d', '#8a7f6d']
}
```

## 3b. Skin: console (built 2026-08-31, verified live)

Technical instrument panel: warm graphite, one amber accent, flat surfaces
(no shadow), 4px radii, and monospace numerals. Reads as a build tool rather
than a report. Values are original.

Fonts: **`'Space Grotesk'`** (body and headings, h1 weight 600,
letter-spacing -.02em) and **`'IBM Plex Mono'`** for KPI values, KPI labels,
the step micro-label and the guide count. The mono is what carries the
character; keep it to numerals and labels, never body copy.

```
:root[data-skin="console"] {
  --page-bg:#f4f4f2; --card-bg:#ffffff; --text:#18181b; --muted:#57534e;
  --border:#dedcd8; --accent:#9a5b06; --shadow:none;
}
:root[data-skin="console"][data-theme="dark"] {
  --page-bg:#0c0c0d; --card-bg:#161618; --text:#ececee; --muted:#8b8b93;
  --border:#282829; --accent:#f0b429; --shadow:none;
}
```

Radii: panels, KPI cards, index cards, media 4px; every control 4px. The
accent is saturated, so the guide box tint is dialled back to 8 percent
light and 9 percent dark instead of the shared 11/17.

```js
palette: {
  colorScheme: 'light dark',
  light: { backgroundColor: '#ffffff', neutralColor: '#18181b', highlightColor: '#9a5b06' },
  dark:  { backgroundColor: '#161618', neutralColor: '#ececee', highlightColor: '#f0b429' },
  colors: ['#c47a05', '#3d84c6', '#d2603f', '#57935f', '#8368cf',
           '#c2559a', '#2f9c92', '#8a8236', '#6e7887']
}
```

Amber leads, steel blue supports. Every colour is mid-tone on purpose: a
brighter amber (#f0b429 for instance) drops under 3:1 on white, so the
series amber is darkened to #c47a05, which clears 3:1 on both grounds while
the lighter amber is used only as the dark-theme highlight.

## 3c. Skin: aurora (built 2026-08-31, verified live)

High key and soft: cool mint-tinted light, deep pine dark, a jade accent,
22px radii and layered soft shadows. The opposite temperature to console and
a rounder shape language than any other skin. Values are original.

Font: **`'Manrope'`** throughout (h1 weight 800, letter-spacing -.028em; KPI
values 700; tool names and the guide heading 700). The heavy geometric
weights are what make it read as friendly rather than corporate.

```
:root[data-skin="aurora"] {
  --page-bg:#eef3f2; --card-bg:#ffffff; --text:#16231f; --muted:#607068;
  --border:#dde7e3; --accent:#0f7d68;
  --shadow:0 2px 6px rgba(20,50,44,.06), 0 12px 28px rgba(20,50,44,.05);
}
:root[data-skin="aurora"][data-theme="dark"] {
  --page-bg:#0a1512; --card-bg:#11201c; --text:#e6f2ee; --muted:#93a9a1;
  --border:#1e332d; --accent:#5fd6b4;
  --shadow:0 2px 6px rgba(0,0,0,.45), 0 12px 28px rgba(0,0,0,.35);
}
```

Radii: panels, KPI cards, index cards, media 22px; guide head 18px; chips
and badges 14px; controls 14px; menu items 10px. Rounding has to be applied
across all of those or the skin looks half-finished.

```js
palette: {
  colorScheme: 'light dark',
  light: { backgroundColor: '#ffffff', neutralColor: '#16231f', highlightColor: '#0f7d68' },
  dark:  { backgroundColor: '#11201c', neutralColor: '#e6f2ee', highlightColor: '#5fd6b4' },
  colors: ['#149574', '#6b6fe0', '#dd6a55', '#3796cd', '#a865cf',
           '#c07f14', '#2c8598', '#6f9e46', '#7c8aa5']
}
```

Jade leads, periwinkle and coral support. Same mid-tone discipline as
console: the luminous mint (#5fd6b4) is a dark-theme highlight only, never a
series colour, because it fails on white.

## 4. Orbit dark mode (no workaround, and never reintroduce one)

Orbit themes its own UI from the page `color-scheme` (verified live
2026-08-31: toolbar background `rgb(24, 23, 28)`, text `rgb(247, 247, 248)`).
So the whole contract is section 1:

```css
:root { color-scheme: light; }
:root[data-theme="dark"] { color-scheme: dark; }
```

Orbit also picks up the surrounding surface, so its chrome shifts with the
skin: in aurora dark the toolbar renders `rgb(17, 32, 28)`, matching that
skin's card colour rather than a fixed grey. Nothing to configure, but worth
knowing when a skin's card colour is chosen.

Earlier versions of this skill claimed Orbit stayed light and prescribed a
`filter: invert(1) hue-rotate(180deg)` block on `.orbit-toolbar` and friends.
That block **inverts the correct dark chrome back to light** and was deleted
from all six demos on 2026-08-31 after light toolbars were reported in the
dark theme. Do not reintroduce it, in any form, without first reading the
computed toolbar colours in both themes.

## 5. Rules for a new skin

A skin touches exactly four places in every page, index.html included: the
Google Fonts link, a `:root[data-skin="NAME"]` CSS block (plus its
`[data-theme="dark"]` twin and any type and radius rules), an entry in the
`SKINS` object, and an `li[data-skin-value]` in the picker. Patch all seven
files in one pass; a skin present in five of them is the kind of drift nobody
notices until someone opens the sixth.

1. Define every property in section 1 for both themes, or ship one theme
   deliberately and say so on the page.
2. Add the skin to the `SKINS` object (font plus palette) and to the picker
   list; the switching code needs nothing else.
3. Verify both themes by reading computed colours, not by eye: chart text,
   card background, and
   `getComputedStyle(document.querySelector('.orbit-toolbar'))` so Orbit's own
   chrome is confirmed to follow. Check any display face at its smallest used
   size. Contrast cannot be reasoned about: 3:1 minimum for series colours,
   and check dense types (treemap, heatmap, packedbubble) per theme.
4. Chart background must follow the card background, or charts look pasted on.
5. Never rely on browser-drawn UI (native select popups) to follow the theme.
6. Keep the layout constants unless a different density is the whole point.
7. Name the skin, state its source and date it.
8. No em dash anywhere. Everything that ships is in English.

A skin is not just a palette. Each of the four differs in temperature, in
type (grotesk, serif display, mono numerals, heavy geometric), in radius (16,
6, 4, 22) and in depth (soft shadow, hairline, flat, layered). A new one that
only changes hue will feel like a duplicate.

## 6. Skin ideas not yet built

- `whitelabel-embed`: neutral grays, customer-accent variable, minimal chrome.
- `blueprint`: cyan on deep navy, hairline grid, drafting-table feel.

Four skins ship: highcharts (cool indigo, 16px, sans), editorial (warm paper,
serif display, 6px), console (graphite and amber, mono numerals, 4px, flat),
aurora (mint and jade, Manrope, 22px, soft depth). A fifth should occupy a
gap none of those cover.

