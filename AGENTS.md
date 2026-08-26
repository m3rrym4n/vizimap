## What this is

A standalone, single-file HTML tool: a Gantt chart for product roadmaps, built with Plotly.js loaded from CDN. No build step, no bundler, no npm dependencies. The entire product is one file — `index.html` — that opens directly in a browser.

## Product boundary

- **Program Increments (PI) and Sprints are overlaid as bands at the top of the chart.** Task activity bars render below them, sharing the same date-based x-axis, so PI/sprint boundaries align visually with the tasks underneath.
- **This is a personal planning tool, not a hosted service.** There is no backend, no deploy target, no container. The deliverable is the file itself.
- Data (Program Increments, Sprints, Tasks) lives as plain JS arrays near the top of the `<script>` block in `index.html`, hand-edited directly — not loaded from an API, not requiring a build step to change.

## Architecture

- Single file: `index.html`. HTML, CSS, and JS all inline.
- Plotly.js via CDN script tag (`https://cdn.plot.ly/plotly-latest.min.js` or a pinned version — pin a specific version once first built, don't float on `latest` long-term).
- Three data arrays define everything to render:
  - `PROGRAM_INCREMENTS`: `[{ name, start, end }]`
  - `SPRINTS`: `[{ name, start, end, piName }]` (piName links a sprint to its parent PI for validation/grouping, not required for rendering)
  - `TASKS`: `[{ name, start, end, status, owner? }]`
- Gantt bars use the standard Plotly technique: `type: 'bar', orientation: 'h'`, date-typed x-axis, `base` = start date, `x` = duration.
- PI and Sprint bands render as their own rows at the top (same technique), with vertical dashed gridlines at PI/sprint boundaries extending down through the task rows below, so alignment is a literal visual gridline, not just proximity.

## Deployment

**None.** No Docker, no container, no CI-driven deploy, no Variflex-managed dev/main promotion in the usual sense — there's nothing to run as a service. CI's only job is a sanity check that the file is well-formed (parses without syntax errors); there's no build or deploy stage after that.

## Constraints

- No frameworks (React, Vue, etc.) — plain HTML/CSS/JS plus the Plotly CDN script only.
- No bundler, no npm install step. If a library is ever needed beyond Plotly, it comes from a CDN `<script>` tag, not a package manager.
- Data arrays are the API. Any change to their shape must be reflected here in this file, since Stan edits them by hand.
- Single branch (`main`). No dev/main split — there's no environment to promote between for a static file with no deploy step. PRs merge directly to `main` once CI (the sanity check) passes.

## Working philosophy (inherited from Pacific Shift Labs' standing rules)

- Boring and simple first.
- Halt and log, don't self-heal — if something's ambiguous or a spec gap is found, stop and report it in the PR/issue rather than guessing silently.
- Automation presents context, it doesn't make product decisions on Stan's behalf.
- One issue worked at a time in this repo — never dispatch multiple concurrently.
- Self-merge is authorized by default once CI is green — PR code review is a post-merge job, not a pre-merge gate.
