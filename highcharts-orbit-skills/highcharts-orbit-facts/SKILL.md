---
name: highcharts-orbit-facts
description: "Verified reference facts for Highcharts and Highcharts Orbit (install, orbit config options, the 20 tool keys, the official Orbit tool set, the exact chart toolbar menu tree, page mode, theming and palette including Orbit's own dark chrome, plus what is explicitly unverified) and the working rules for them. Use whenever writing or reviewing Highcharts or Orbit configuration, building an Orbit demo, or answering a question about Orbit tools, menu paths, page mode or Highcharts theming. Hosting, serving, allowlist and deploy issues live in the separate highcharts-orbit-hosting skill."
---

# Highcharts and Orbit: verified reference

Standing rules plus facts verified 2026-08-29/31 against the official
sources and live browser tests. Where this file and a live source disagree, the
live source wins. Hosting, serving, the CDN Referer requirement, the Orbit
referrer allowlist and deploy traps live in the separate
`highcharts-orbit-hosting` skill; design profiles and the demo template
contract in `highcharts-orbit-design`; general working traps and verification
habits in `highcharts-orbit-lessons`.

## 1. Working rules

- **Concise and fact-based.** Bullet lists with exact URLs, paths and values.
  No preamble, no unsolicited caveats. Verify, then state.
- **Never speculate.** Look up current docs or test empirically before stating
  anything. A green build does not prove an option exists.
- **Never use an em dash (U+2014)**, in any text, any language, any context.
- **Flag what was not verified** rather than presenting everything at equal
  confidence.
- **Always run chart choices through the Chart Chooser MCP and Highcharts Dev
  MCP** when they are connected: `recommend_chart` / `cc_detect_chart_type` for
  type choices, `validate_config` for configs. Known MCP limitations:
  - `validate_config` flags per-series-item options (`data`, `pointStart`,
    `pointInterval`, `tooltip`, `dashStyle`, `opacity`, `fillOpacity`,
    `marker`, `innerSize`, `layoutAlgorithm`, `colsize`) and top-level
    `palette` as "Unknown option". These are real, documented options; treat
    those warnings as false positives and confirm on api.highcharts.com.
  - `get_chart_type_info` can name a wrong module: it reported
    `modules/packedbubble.js`, which does not exist on the CDN. Packedbubble
    ships in `highcharts-more.js` (verified 2026-08-30 after a live error #17).
    Confirm every module path against code.highcharts.com (with an http(s)
    Referer, see the hosting skill) before shipping.
- Verified module homes for the demo chart types (2026-08-30, all loaded and
  rendered live): pie/donut in core; treemap in `modules/treemap.js`; heatmap
  in `modules/heatmap.js`; streamgraph in `modules/streamgraph.js`; histogram
  in `modules/histogram-bellcurve.js`; candlestick in `modules/stock.js`;
  boxplot, gauge, bubble, arearange and packedbubble in `highcharts-more.js`.

## 2. Sources of truth

| Topic | Source |
|---|---|
| Highcharts concepts and module requirements | https://www.highcharts.com/docs/index |
| Highcharts option reference, one URL per option | https://api.highcharts.com/highcharts/ |
| Working demos | https://www.highcharts.com/demo |
| Sample code | https://www.highcharts.com/samples |
| Orbit machine-readable reference | https://orbit.highsoftlabs.com/llms-full.txt |
| Orbit live demos | https://orbit.highsoftlabs.com/demos/ |
| Orbit changelog | https://orbit.highsoftlabs.com/changelog/ |
| Orbit tokens and signup | https://orbit.highsoftlabs.com/#signup |
| Claude plugin marketplace | https://orbit.highsoftlabs.com/claude/marketplace.json |

Use the Highcharts and the Orbit sources together, never one instead of the
other. Fetch the exact option URL and read the "Requires" note for the module.
No Stack Overflow, no blog posts, no training memory. If a source cannot be
fetched, say so. Note: fetching code.highcharts.com requires a Referer header
(see the hosting skill). The Orbit portal docs under `/portal/docs/` redirect
to a login; `llms-full.txt` is the public equivalent and is what to cite.

For anything about the running Orbit build (menu labels, panel contents, where
a tool lives) the live DOM outranks every document, including this file.

## 3. Orbit install

Orbit adds an analysis toolbar to Highcharts charts, maps and grids. House
rule: every chart in a demo gets Orbit, never plain Highcharts.

```html
<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/modules/exporting.js"></script>
<script src="https://code.highcharts.com/modules/annotations.js"></script>
<!-- Orbit always last -->
<script src="https://orbit.highsoftlabs.com/module/<API-TOKEN>/orbit.js"></script>
```

- Not on npm, code.highcharts.com, cdnjs, jsdelivr or unpkg.
- Loads **last**, after Highcharts core and every Highcharts module used. It
  wraps Highcharts internals and fails silently otherwise.
- Self-initializes and reads the key from the URL. No `configure()` call for
  authentication.
- The installation token is client-side, not a secret: it appears in the public
  script URL. It is still scoped by the installation's referrer allowlist
  (domain lock). Serving requirements, the code.highcharts.com Referer/403
  behavior, the file:// trap, localhost serving and allowlist verification:
  `highcharts-orbit-hosting`.
- `exporting.js` is required for the Export tool, `annotations.js` for the
  Annotate (Draw) tool, both on by default. `highcharts-more.js` when the
  chart type needs it. Stock indicator modules enable the indicators panel.

Opt a chart in with an `orbit: { enabled: true }` (or `id` in page mode)
block. The script tag alone does nothing; the orbit block alone does nothing.

## 4. Orbit config

Per-chart `orbit` object:

| Option | Meaning |
|---|---|
| `enabled` | Opts this chart in. Not needed in page mode when `id` is set |
| `id` | Stable content id. Required for page mode |
| `tools` | Whitelist of tool keys. Omit to show everything the installation offers |
| `initialTool` | Opens that tool on load. Must also be in `tools` if a whitelist is set |
| `menuVisibility` | `'always'` (default), `'auto'`, `'compact'` |
| `allowToolPopout` | Default true. Panels detach into movable windows |
| `allowPinning` | Page mode only. Pin a filter or compare state |
| `toolbarTarget` | Element id. Renders the toolbar into your own header bar |
| `toolPaneTarget` | Element id. Opens panels into your own inspector column |
| `llmContext` | `{ htmlNodes: [ids], text: [strings] }` extra context for the AI tools |
| `relationships` | Declared cross-content links, used by page mode |

The whitelist accepts **exactly 20** keys, verbatim:

```
summary, correlations, distribution, kpi, contribution, anomaly,
control-limits, forecast, trendline, insights, narrate, ai, filter,
derived, altviz, fullscreen, history, export, annotate, share
```

There is no `grid`, `compare`, `zoom` or `indicators` key. The Data Grid view
and the Stock indicators panel appear automatically when they apply.

### 4b. The official Orbit tool set (product descriptions)

**Demos and their data MUST always be shaped so these features have something
real to show.** This is a standing requirement, not a nice-to-have. Per tool:

- **Anomaly Detection**: scans the data for outliers; flagged points highlight
  directly on the chart. Data needs a stable baseline plus deliberate spikes.
- **Forecast**: projects trends forward; fit scores tell how much weight to
  put on each projection. Data needs a visible, extrapolatable trend.
- **Correlations**: Pearson coefficient between series, shown as labeled bars
  with strength ratings. Needs 2+ series that genuinely co-move (and ideally
  one pair that does not).
- **Summary Stats**: min, max, mean, median, standard deviation and trend
  direction per series. Works on anything; spreads should be meaningful.
- **Data Grid**: chart data as an interactive table (Highcharts Grid Pro).
  Point names/categories should read well as table rows.
- **Filter & Focus**: toggle series on/off and adjust axis ranges. Needs
  multiple series worth isolating.
- **Present**: builds a one-click slide deck presented alongside the chart.
  Titles, subtitles and llmContext should carry the story.
- **Derived Series**: compute new series from existing data. Series whose
  ratio/difference is meaningful make this land.
- **Indicators**: Highcharts Stock technical indicators on any chart. Load the
  indicator modules; OHLC/candles show it best.
- **AI Assistant** (AI): ask questions about the data or modify the chart via
  prompts. llmContext is what makes answers specific.
- **Narrator** (AI): copy-paste-ready summaries in four tones: executive,
  technical, casual, presentation-ready.
- **Insights** (AI): reads the data, returns a summary plus directions worth
  exploring. Needs a real finding in the data.
- **Alt Visualization** (AI): suggests alternative chart types with an
  explanation of what each would showcase.
- **Full Screen**: expands the chart to fill the screen.
- **Chart**: the plain chart view, front and center, no side panel.

Where each tool sits in the toolbar: the exact menu tree, with the real labels
and descriptions read from the live DOM, is section 5a. Use it whenever demo
copy names a menu path.

### 4c. Page mode

`enabled: true` on several charts does **not** link them; only
`Highcharts.orbitPage()` does. Give each content a stable `id`, then call it
after the charts exist:

```js
Highcharts.orbitPage({ mode: 'augment', relationships: { enabled: true } });
```

Contents link on identical category strings or a shared datetime axis
(`relationships.dateToleranceMs`, 1 hour default). Never hand-write
cross-filter JavaScript. Page-scope whitelist:
`orbitPage({ tools: { only: [...] } })`; whole-page sharing is `page-share`.

**Date filters, verified 2026-08-29/30, two hard facts:**

1. The page-mode Filters panel date presets (Today, Last 7d/30d/90d, This
   year, All) are **relative to the CURRENT calendar date**, not the data's
   extent. Anchor datetime dummy data so the last point is today, or every
   preset selects an empty range.
2. **The date filter only works when every point carries an explicit x
   timestamp** (`[x, y]` pairs). Series positioned with
   `pointStart`/`pointInterval` render fine unfiltered but produce an EMPTY
   filtered view for any date range (A/B verified on the deploy domain).
   Always emit explicit timestamps:
   `data.map(function (v, i) { return [START + i * DAY, v]; })`.

When a date filter applies, Orbit renders a separate filtered view of each
chart inside its container; the original chart object keeps its full extremes.

Maps hide the axis-based tools (forecast, trendline, anomaly, control-limits,
correlations, filter, derived, altviz); they cannot be whitelisted back in.

Orbit is beta; check the changelog.

## 5. Theming

- `palette` is a real top-level option (`colors`, `colorScheme`, `light`,
  `dark`, `injectCSS`). `colorScheme: 'light dark'` switches on system
  preference or a `.highcharts-light`/`.highcharts-dark` class on a parent; no
  chart recreation (verified live, Highcharts 13.0.2).
- **Orbit's own toolbars and panels DO follow the page `color-scheme`**
  (verified live 2026-08-31, beta build of that date). With
  `:root[data-theme="dark"] { color-scheme: dark; }` the Orbit chrome renders
  itself dark: toolbar background `rgb(24, 23, 28)`, text `rgb(247, 247, 248)`,
  and the page menubar and drawers follow. Nothing else is needed: set
  `color-scheme` and stop.
  An earlier version of this skill claimed the opposite, and the demos carried
  a CSS `filter: invert(1)` workaround because of it. That claim is **obsolete
  and the workaround is harmful**: it flips the now-correct dark chrome back to
  light. It was deleted from all six demos on 2026-08-31. Never reintroduce it.
  Verify with `getComputedStyle(document.querySelector('.orbit-toolbar'))` in
  both themes rather than by eye.
- `styledMode: true` removes presentational attributes; default stylesheet
  `css/highcharts.css`. JS themes apply only to charts created after
  `setOptions` and force recreation plus Orbit re-attach; avoid for pure color
  switching.
- Verify contrast in every theme shipped: 3:1 minimum for series colors.
- Design tokens and profiles: `highcharts-orbit-design`.

## 5a. Chart toolbar menu tree (read from the live DOM, 2026-08-31)

Labels and descriptions below are exact; `>` marks a submenu. Read with
`document.querySelectorAll('.orbit-toolbar__menu')` plus
`.orbit-toolbar__dropdown-label` / `.orbit-toolbar__dropdown-desc`. Re-read
after any Orbit update rather than trusting this copy.

- **View**: Chart | Fullscreen "Expand to fill the screen" |
  Alt. Visualization "Try different chart types" | Representations >
  (Summary "View the data as a per-series statistics table" |
  Distribution "View the data as a histogram" |
  KPIs "View the data as KPI tiles")
- **AI**: AI Assistant "Chat with your data" | Insights "AI analysis of your
  data" | Narrator "Copy-paste summaries in any tone"
- **Analyze**: Relationships > (Correlations "How series move together" |
  Contribution "Share of total (Pareto)") | Trends > (Trend Line "Fit a line
  to existing data" | Forecast "Project trends forward" | Indicators
  "Technical indicators from Highcharts Stock") | Quality > (Anomaly Detection
  "Find statistical outliers" | Control Limits "Process-control bands")
- **Annotate**: Draw "Draw shapes and annotations on the chart"
- **Transform**: Filter & Focus "Toggle series and filter by value" |
  Derived Series "Compute new series from data"
- **Share**, **Export**: direct buttons, no menu.

Trigger offsets in the 1210px toolbar (left px, width px): View 8/87,
AI 97/72, Analyze 170/104, Annotate 276/111, Transform 389/117, Share 509/74,
Export 584/78. Bar height 38px. Useful when compositing screenshots.

Page mode adds a top menubar (1450px wide, 48px tall in the demos):
Share | Highlights | AI > (Insights, Chat) | Advanced > (Relationships, Page
Context) | Search tools | Data | Pinned | Filters | Compare. Filters opens a
right drawer with DATE (free from/to plus presets Today, Last 7d, Last 30d,
Last 90d, This year, All), per-series SERIES check dropdowns and
"Clear all filters". Compare opens a drawer titled Compare with a
"COMPARE BY" combo, "Choose a dimension".

Consequence for demo copy: a how-to line must name the real path. Contribution
is Analyze > Relationships > Contribution, not a top-level tool; Distribution
is View > Representations > Distribution; Indicators is Analyze > Trends >
Indicators, not a panel on the chart.

## 6. Next.js integration

Global script tags in the root layout (core, modules, Orbit last); never the
npm `highcharts` package alongside, never `next/script` for these. Charts from
`window.Highcharts` in client components after mount; destroy in cleanup.
`Highcharts.orbitPage()` once per page after the charts exist. Existing
production integrations follow exactly this pattern.

## 7. Demo data rules

Dummy data only. 12 to 90 points per series, 1 to 4 series, inline literals,
no fetch, no build step. Plausible magnitudes, generic names. Deterministic:
seed any generator and paste the output; never `Math.random()` at runtime.
Derived structures (boxplot five-number summaries, heatmap triplets, OHLC by
formula from closes) may be computed at load time from the literals.

**Shape the data for the full official tool set in 4b, always.** In practice
every demo needs: a stable baseline with deliberate outliers (Anomaly, Control
limits), a clean extrapolatable trend (Forecast), 2+ genuinely co-moving
series (Correlations), meaningful spreads (Summary Stats), multiple series
worth isolating (Filter & Focus), series with meaningful ratios (Derived),
and titles/subtitles/llmContext that carry the story (AI tools, Present).

**Datetime axes, two absolute requirements (section 4c):** anchor every series
so the last point is today, and give every point an explicit x timestamp.
Write in-page dates relative to the window, never as fixed calendar dates.

**Chart type variety:** one demo should not be five line charts. Composition
to donut, treemap or packedbubble; distribution of raw category values to
boxplot; distribution of one variable to histogram; category-by-time intensity
to heatmap; additive flow over time to stacked area or streamgraph; a single
reading against bands to a gauge; OHLC to candlestick (which also brings the
indicators panel). Load the right module (section 1), keep Orbit last.

## 8. Still unverified, do not assert

- Page-mode tool keys other than `page-share`.
- `configure({ pageKey })`, `orbitPage({ autofilter, layout, pageKey })`.
- Whether Orbit needs re-attaching after a chart is destroyed and recreated.
- The exact key mapping for the Present, Data Grid and Full Screen UI names
  onto the 20-key whitelist (Present may be `history` or `share` adjacent;
  not confirmed).
- Which tools appear on non-cartesian kinds (pie/donut, treemap, packedbubble,
  gauge): toolbars attach, per-kind tool sets not inventoried.
- Whether Orbit's dark chrome is complete inside popouts, the command palette
  in use, and streaming AI panels (the toolbar, menubar and drawers are
  verified dark).
- Theme CSS/JS paths under `css/themes/` and `themes/`.

