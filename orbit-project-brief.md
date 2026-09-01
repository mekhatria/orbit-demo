# Orbit demo project — kickoff brief for Claude Code

Context for whichever Claude instance picks this up in Claude Code. Read this
first, then read the skills below in full before writing any code.

## Goal

Build my own Highcharts Orbit demo site (like a colleague's
`highcharts-orbit.vercel.app`), using the skills he shared. Static, no
backend, deployed to Vercel (or Netlify/GitHub Pages).

## Skills to install first

Unzip these into `.claude/skills/` (project-level) or `~/.claude/skills/`
(personal, available everywhere) before starting:

- `highcharts-orbit-facts` — install steps, config options, the 20-key tool
  whitelist, chart toolbar menu tree, theming/dark-mode facts
- `highcharts-orbit-design` — page template contract, the four named skins,
  guide-box slider spec, screenshot pipeline
- `highcharts-orbit-lessons` — traps, verification methods, pre-ship checklist

**Missing: `highcharts-orbit-hosting`.** All three skills above repeatedly
reference a fourth skill covering the referrer allowlist mechanics, the
CDN Referer/403 requirement, the `file://` trap, localhost serving, and
deploy traps — that one was never shared with me. **Get it from the
colleague before doing any hosting/deploy work.** Until then, treat hosting
setup as unverified and re-check everything against the live deployed site
rather than assuming.

## Before writing any code

1. Sign up for my own Orbit token at `https://orbit.highsoftlabs.com/#signup`
   — do not reuse the colleague's token.
2. Register my deploy domain(s) on that token's referrer allowlist
   (mechanics TBD until the hosting skill arrives — ask me for the token
   once obtained, I'll paste it in, don't hardcode anyone else's).
3. Confirm these MCP servers are connected in this Claude Code session
   (I already use them in claude.ai chat, may need connecting here too):
   - Highcharts Chartchooser MCP — `https://chartchooser-mcp.highcharts.ai/mcp`
   - Highcharts Developer MCP — `https://mcp.highcharts.ai/developers/mcp`
   - Highcharts Export MCP — `https://mcp.highcharts.ai/export/mcp`
   Per the lessons skill, run every chart type choice through
   `cc_detect_chart_type` / `recommend_chart` and every config through
   `validate_config` before writing it into a file, not after.

## Working decisions already made

- **Hosting**: static site, deployed to Vercel. No build step unless we
  decide to go Next.js (pattern is in the facts skill section 6 if so).
- **No real data**: dummy inline literals only, per the facts skill's demo
  data rules (12–90 points/series, deterministic, shaped for the specific
  Orbit tools each demo is meant to showcase).
- Follow the pre-ship checklist in the lessons skill (section 6) before
  calling any demo done — script load order, tool-key validity, module
  loading, theme contrast, menu-path accuracy in the how-to copy, no em
  dashes.

## Open questions to raise with me, not guess at

- Which of the four skins (highcharts / editorial / console / aurora) to
  start with, or whether to build the fifth "gap" skin mentioned in the
  design skill.
- How many demos, what industries/data stories they should tell.
- Whether to mirror the colleague's six-demo structure or start smaller
  with one demo end-to-end (get toolbar + allowlist verified working
  before multiplying pages).
