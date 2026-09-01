---
name: highcharts-orbit-lessons
description: "Hard-won lessons, traps and a pre-deploy checklist from building live Highcharts Orbit showcases: silent degradation, script load order, verification methods that actually prove a claim, the Chart Chooser and Highcharts Dev MCP workflow, data-handling traps with their symptoms, auto-insight quality rules, and what makes a demo land. Use before starting, debugging or shipping any Orbit demo, and whenever a chart renders but the Orbit toolbar or a tool is missing. Hosting, serving and deploy specifics live in the separate highcharts-orbit-hosting skill."
---

# Learnings and reminders

Collected from building live Orbit showcases, generalized so it applies to any
Orbit demo. Sources, methods, tools and traps. Nothing here is project-specific.
Every entry is either something that actually went wrong or something that
actually proved a claim. Read section 1 before starting, section 6 before
saying "done". Hosting, serving and deploy specifics (CDN Referer requirement,
file:// trap, localhost serving, Orbit allowlist, deploy traps) live in the
separate `highcharts-orbit-hosting` skill.

## 1. The five that cost the most time

1. **Silent degradation looks exactly like success.** Without a valid referrer
   the Orbit module does not load, charts still render, and the build is green.
   Nothing in a log tells you. Always check for the toolbar on the deployed
   domain (details in the hosting skill).
2. **Script load order is not negotiable.** Highcharts core, then modules, then
   Orbit, then your chart code. Orbit wraps Highcharts internals and fails
   silently if Highcharts is not there yet. A framework's script loader
   (`next/script` for example) reorders this and breaks it.
3. **A green build proves nothing.** An option that does not exist is ignored
   without an error, and a series type whose module was never loaded just does
   not draw.
4. **Never guess a field or parameter name.** A plausible-looking but wrong
   parameter is silently ignored by most APIs, which then returns the wrong
   slice of data with a 200. Call the endpoint once and read the actual
   response before writing any code against it.
5. **Restating a stale caveat is the same mistake as guessing.** A limitation
   that was fixed since it was last seen must be re-tested before it is
   repeated.

## 2. Verification methods that work

The rule: proof comes from the deployed artifact, not from local state.
Hosting-specific proofs (live URL grep, allowlist curl, deployment status) are
tabulated in the hosting skill.

| Claim | What actually proves it |
|---|---|
| This chart type is the right one for this data | Chart Chooser MCP (`cc_detect_chart_type`, `cc_choose_viz_type`) or Highcharts Dev MCP `recommend_chart`, not taste |
| This config is valid | Highcharts Dev MCP `validate_config`, then the option's own api.highcharts.com page for anything it flags |
| This series type needs this module | The "Requires" note on the option page. `get_chart_type_info` has named a module that does not exist |
| The chart looks right in both themes | Open it and look, or screenshot it. Contrast cannot be reasoned about |
| Orbit's toolbar follows the theme | Read its computed background and colour in both themes. A memory of "Orbit has no dark mode" was wrong for months and produced a harmful invert hack |
| A menu item lives where the demo says | Read the live menu DOM. Menu paths drift between beta builds and were wrong in the first screenshot set |
| An option exists | The option's own page on api.highcharts.com, including its "Requires" note |
| A tool key is valid | The 20-key list in the Orbit reference. Nothing else |
| A filter or preset behaves as assumed | Apply it in a live browser and read the resulting state, never assume its semantics |

Supporting habits:

- Server-render the page text where the stack allows it, purely so the deployed
  result is verifiable by fetching HTML. This paid for itself when browser
  automation was unavailable.
- Keep a pre-deploy health check script: verify parameter names against the
  provider's own schema endpoint, ping every source, check that any manifest
  matches the files it lists.
- Compare rendered screenshots when choosing between visual candidates. A
  favicon drawn from a detailed logo only survived at 32 and 64 px after being
  redrawn on a coarse grid, and that was decided by looking at renders, not
  sources.
- Write every bug found into a lessons file. Promote the recurring ones to a
  numbered decision, so the rule survives and not just the patch.

## 3. Tools and commands

**The two MCPs come first on anything chart-shaped.** Run every chart type
choice and every config through them before writing the file, not after:

- **Chart Chooser MCP**: `cc_detect_chart_type` and `cc_choose_viz_type` to
  check that the chart matches the question the data answers. Use it
  especially when reaching for a less common type (streamgraph, packedbubble,
  boxplot) so variety stays defensible rather than decorative.
- **Highcharts Dev MCP**: `recommend_chart` for type choices,
  `validate_config` on every config, `search_docs`, `search_snippets` and
  `get_chart_type_info` for lookups.

Both have known limits, documented in `highcharts-orbit-facts` section 1:
`validate_config` reports real per-series options and top-level `palette` as
"Unknown option" (false positives, confirm on api.highcharts.com), and
`get_chart_type_info` has named a module path that does not exist on the CDN.
Treat the MCPs as a fast first pass, and the official docs as the authority
whenever the two disagree.

Everything else:

- **A Linux sandbox shell** does nearly everything: git, package scripts,
  migrations, ad hoc Node and Python scripts.
- **`curl` patterns worth keeping:** call an API once with each candidate
  parameter name and diff the responses, to find the real one. The hosting
  curl patterns (live page grep, module URL with Referer) are in the hosting
  skill.
- **GitHub CLI or API:** dispatch a workflow manually to run long jobs on
  runners.
- **Spreadsheet tooling** (openpyxl-based) for any backlog or matrix that has to
  survive outside the repo.
- **PIL (Pillow)** for building UI screenshots: see the screenshot pipeline in
  `highcharts-orbit-design` section 1c. html2canvas cannot paint Orbit's portal
  menus, and it hangs outright on `color-mix()` ("unsupported color function"),
  so capture the bare toolbar with a flat-colour override stylesheet injected
  and composite the menus.

Two environment facts:

- **Background processes do not survive between separate shell calls.** A long
  job must be a dispatched CI workflow, not a backgrounded command.
- **CI runners have their own IPs and fresh quotas**, which is the escape hatch
  when a sandbox IP hits a daily API limit.

## 4. Traps, with the symptom

Grouped so they can be scanned. Symptom first, because that is how you meet
them. Hosting and deploy traps (403s, file://, blocked deploys, caching) are in
the hosting skill.

### Orbit and Highcharts

- **Charts render, no toolbar.** Referrer allowlist, or Orbit loaded before
  Highcharts, or the hosting `Referrer-Policy` strips the origin (hosting
  skill).
- **Several charts have toolbars but do not filter each other.** That is
  expected. Per-chart `enabled: true` does not link anything. Use
  `Highcharts.orbitPage()`. Never hand-write cross-filter JavaScript.
- **A tool is missing from the toolbar.** Either it is excluded by a whitelist,
  or it is `share` (off by default), or it needs a module that is not loaded, or
  it does not apply to that content kind (maps hide all axis-based tools).
- **`initialTool` does nothing.** It is not in the `tools` whitelist.
- **A series type does not draw and the console shows error #17.** The module
  is missing, or loaded from a path that does not exist. Confirm the module
  home on code.highcharts.com rather than trusting a lookup tool.
- **The page-mode date presets (Today, Last 7d/30d/90d) seem dead.** They are
  relative to the current calendar date, not to the data's extent. Demo data
  with a fixed historical end date gives an empty selection. Anchor datetime
  demo data so the last point is today (dynamic `pointStart`, literal values)
  and write in-page dates relative to the window.
- **Orbit's toolbars look light while the page is dark.** Do not reach for a
  filter. Orbit themes itself from `color-scheme`; a light toolbar in a dark
  page means either `color-scheme: dark` is not set on the root, or some CSS
  (historically an `invert(1)` "workaround") is flipping the correct dark
  chrome back. Read the computed colours before touching anything.
- **Map zoom buttons keep library default colors.** They are themed separately
  via `mapNavigation.buttonOptions.theme`, and the theme has to be reapplied on
  every theme switch as well as at creation.
- **Treemap and dense labels become unreadable in one theme.** Label contrast
  needs explicit work per theme, it is not inherited.
- **Charts sit flush against each other.** Use one shared stack class everywhere
  rather than per-page spacing, then audit every page for deviations. What
  worked: 18px between charts, 14px after section text, 30px between sections.

### Data handling (applies when a demo uses real data rather than dummy data)

- **Empty chart, no error.** A renamed timestamp field made the parser produce
  `new Date(null)`, so every row landed on epoch 0.
- **History collapses to a handful of points.** A prune-older-than-N-years job
  that ignored cadence wiped annual and monthly series. Prune only sub-daily
  cadences.
- **An ancient year collides with a modern one.** `1-01-01` is read as 2001 by
  Postgres. Zero-pad years.
- **A precompute job dies quietly.** The statement timeout applied through the
  REST layer. Set the timeout on the database function itself.
- **A headline count renders as 0.** An exact row count over millions of rows
  exceeded the anonymous role's timeout and fell back to 0. Use the planner
  estimate for hero numbers.
- **Disjoint time windows.** A REST layer capped rows per request (1000 by
  default in PostgREST). Paginate explicitly.
- **429 on a big backfill.** In order of what worked: chunk the range, throttle
  to a fixed calls-per-minute budget, make the script resumable with per-entity
  upsert, then move the long run to CI runners.
- **The first matching station or entity is dead.** Try up to four candidates
  sorted by a recency field. Skip endpoints that return 408 rather than
  retrying.
- **Values are valid but wrong-looking.** A field named as if it were a founding
  date held the date of the current regulation. No schema check catches
  semantic mismatch. Only the provider's docs do.
- **A default selection is silently empty.** Check coverage before picking any
  default, and default to the best-covered period.

### Auto-insight quality

These generalize to any demo that shows Orbit's analytical tools, and to any
insight feature built alongside them.

- **Records that are not records.** Using `>=` flagged days that merely touched
  a series floor. Require strictly beating history by a margin of at least 5
  percent of the p10 to p90 spread, and skip zero-variance series.
- **Twin pairs at r near 1.** Same metric, same region, two sources. That is
  validation, not insight. Filter out same-metric-stem plus same-region pairs.
- **Percentiles comparing unlike things.** An instantaneous reading against a
  daily mean. Compare like with like.
- The governing principle: **if it would not surprise a reader who knows the
  series, do not show it.**

## 5. What makes an Orbit demo land

- The tools that make Orbit worth showing (anomaly, forecast, correlations,
  control limits, indicators) need series that actually move. Flat data makes
  the product look like nothing.
- One capability per demo. State in one sentence what to click.
- `llmContext` is not optional in practice. It is what lets the AI tools mention
  the caveat or the KPI that the series alone cannot show. Treat it as a
  required field in the template.
- `data-orbit-context` on KPI cards and note blocks is the cheap version of the
  same thing, and it needs no config.
- `Highcharts.orbit.context.getAll()` returns everything currently captured as
  page context. Use it to debug what the assistant can and cannot see.
- Keep the module URL in one config file that also exposes the shared defaults
  helper, the loader and the error display. Every demo imports that, so the
  token and the load order exist in exactly one place.
- Record source and license as per-chart metadata even for dummy data, so the
  footer pattern is real and reusable.
- Chart variety only helps when each type is the right answer to its own
  question. Run the choice through the Chart Chooser MCP; a streamgraph picked
  for looks reads as decoration, the same one picked for additive flow over
  time reads as competence.
- A how-to box teaches only if its instructions are true. Every "open X, then Y"
  line must match the live menu tree, and the screenshot beside it must
  highlight the row it names. Wrong paths are the fastest way to make a demo
  feel unfinished.

## 6. Pre-ship checklist

1. Every chart type was checked against the Chart Chooser MCP or
   `recommend_chart`, and every config passed `validate_config` with any
   flagged option confirmed on api.highcharts.com.
2. Script order: core, modules, Orbit, demo code. Orbit is last.
3. Every Orbit tool key used is one of the 20. `share` is listed explicitly if
   wanted.
4. Every required module is loaded for every tool on the toolbar, with each
   module path confirmed against code.highcharts.com.
5. Every Highcharts option used has been read on api.highcharts.com.
6. Dummy data is deterministic, small, shaped for the tool being shown, and
   datetime series are anchored so the last point is today.
7. Theme toggle checked in every theme, including contrast and any element that
   styled mode leaves unstyled. Read Orbit's own chrome with
   `getComputedStyle(document.querySelector('.orbit-toolbar'))` in both themes:
   it follows `color-scheme` on its own, so any CSS that tries to force it
   (an `invert` filter above all) is a bug, not a workaround.
8. Every how-to line in the guide box names a real menu path, checked against
   the menu tree in `highcharts-orbit-facts` section 5a, and each slide's
   screenshot highlights the tool that slide teaches. Audit it as a table of
   tool name against shot filename before shipping; mismatches here are
   invisible in a build and obvious to a viewer.
9. Guide box screenshots are all one size (1200x400 PNG), cropped tight. If a
   shot has a wide white margin, the crop step was skipped.
10. No em dash anywhere.
11. The hosting checklist in `highcharts-orbit-hosting` has been run (allowlist,
    Referrer-Policy, served over http(s), live page opened and toolbar seen).

