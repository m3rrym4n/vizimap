## What this is

A standalone, single-file HTML tool: a Gantt chart for product roadmaps, built with Plotly.js loaded from CDN. No build step, no bundler, no npm dependencies. The entire product is one file — `index.html` — that opens directly in a browser.

## Product boundary

- **Program Increments (PI) and Sprints are overlaid as bands at the top of the chart.** Task activity bars render below them, sharing the same date-based x-axis, so PI/sprint boundaries align visually with the tasks underneath.
- **Milestones are single-point-in-time markers** (a date, not a range), rendered as diamond markers on their own dedicated row between the Sprint band and the Lane/task rows. Independent of Lanes/PIs/Sprints — no parent relationship, unless revisited later.
- **Tasks are organized into named Lanes (swimlanes), not one row per task.** All tasks assigned to the same lane render on that lane's single row. This is a deliberate change (2026-08-26) from the original one-row-per-task model. Lanes are independently manageable (add/edit/delete/reorder), separate from tasks themselves.
- **Tasks carry a longer freeform description, surfaced by clicking their bar.** The existing hover tooltip stays as a quick-glance summary (name/dates/status/owner); clicking a task bar shows an on-chart overlay with its full description — a separate, more deliberate interaction (2026-08-26).
- **This is a personal planning tool, not a hosted service.** There is no backend, no deploy target, no container. The deliverable is the file itself.
- **This is a real roadmap-authoring tool, not just a static viewer.** The product owner creates, edits, and plans roadmaps directly in the tool — adding/editing/removing Program Increments, Sprints, Lanes, Tasks, and Milestones through an in-app UI — to produce a clean visual to brief to leadership. This is a deliberate architecture decision (2026-08-26), not the original scope: the tool started as "hand-edit the JS arrays," and has grown into "open a roadmap file, edit it in the app, save it back."
- **Two distinct view modes: Chart and Configure.** A toggle at the top of the page shows either the chart (plus PNG export) or the editing/configuration UI (all entity forms/lists, plus Open/Save) — never both at once. Having both visible simultaneously was explicitly rejected as messy (2026-08-26).
- **Exportable to PNG** for dropping directly into briefs/slide decks (see #13).

## Architecture

- Single file: `index.html`. HTML, CSS, and JS all inline.
- Plotly.js via CDN script tag, pinned to a specific version (`2.35.2` as of this writing) — don't float on `latest`.
- Data collections — same shapes whether they come from the built-in sample data or a loaded file:
  - `PROGRAM_INCREMENTS`: `[{ name, start, end }]`
  - `SPRINTS`: `[{ name, start, end, piName }]` (piName links a sprint to its parent PI for validation/grouping, not required for rendering)
  - `MILESTONES`: `[{ name, date }]` — single date, not a range. Independent entity, no parent reference.
  - `LANES`: `[{ name }]` — display order in this array is the display order in the chart (top to bottom). No separate `order` field; reordering means reordering this array.
  - `TASKS`: `[{ name, start, end, status, owner?, lane, description? }]` — `lane` references a `LANES` entry's `name`, same referencing convention as `SPRINTS.piName`. `description` is optional freeform text, distinct from `name` (short label) — surfaced via click-to-overlay, not the hover tooltip.
- Gantt bars use the standard Plotly technique: `type: 'bar', orientation: 'h'`, date-typed x-axis, `base` = start date, `x` = duration. Milestones use a scatter trace with `marker.symbol: 'diamond'` instead, since they're a point in time, not a range.
- PI and Sprint bands render as their own rows at the top (same technique), with vertical dashed gridlines at PI/sprint boundaries extending down through the task rows below, so alignment is a literal visual gridline, not just proximity.
- **Task rows are one-per-lane, not one-per-task.** Multiple tasks assigned to the same lane share that lane's row. **Deliberate scope boundary:** if two tasks in the same lane have overlapping date ranges, they will visually overlap on the chart — this is accepted as useful signal (a real scheduling conflict), not a bug. Automatic sub-row packing/offsetting for overlapping same-lane tasks is explicitly out of scope unless revisited later.
- **Click vs. hover on task bars serve different purposes:** hover (native Plotly tooltip) = quick glance (name/dates/status/owner, unchanged since #5/#6). Click = deliberate, shows a dismissible on-chart overlay with the task's full `description`. These are separate interactions, not a replacement of one by the other.

## Persistence (decided 2026-08-26)

- **Mechanism: the File System Access API** (`window.showOpenFilePicker` / `showSaveFilePicker`). "Open Roadmap" picks a JSON file from disk; edits happen in memory; "Save" writes back to that same open file handle — no server, no download-then-manually-replace-the-file step. This is the intended day-to-day workflow: the JSON file sits next to `index.html` like a companion data file, opened and saved in place.
- **Chromium-only by design, not an oversight.** The File System Access API only works in Chrome/Edge, not Firefox or Safari. Confirmed 2026-08-26: Stan only uses Chrome/Edge, so **no fallback for other browsers is required.** Don't build one — it would be unused complexity. If the API is unavailable, a clear error message is enough; no need for a degraded-but-working alternate path.
- **First-run behavior:** if no file has been opened yet, the app falls back to the built-in sample data already in the file — so `index.html` still opens and shows something meaningful with zero setup, same as today.
- **JSON schema on disk** should be a single object wrapping the five collections: `{ programIncrements: [...], sprints: [...], milestones: [...], lanes: [...], tasks: [...] }` — matches the in-memory shapes above, one file, no separate files per collection. A file missing a given key (e.g. saved before `milestones` existed) should load as an empty array for that key, not error.

## Deployment

**None.** No Docker, no container, no CI-driven deploy, no Variflex-managed dev/main promotion in the usual sense — there's nothing to run as a service. CI's only job is a sanity check that the file is well-formed (parses without syntax errors); there's no build or deploy stage after that.

## Constraints

- No frameworks (React, Vue, etc.) — plain HTML/CSS/JS plus the Plotly CDN script only.
- No bundler, no npm install step. If a library is ever needed beyond Plotly, it comes from a CDN `<script>` tag, not a package manager.
- The JSON schema above is the API once persistence lands. Any change to its shape must be reflected here.
- Single branch (`main`). No dev/main split — there's no environment to promote between for a static file with no deploy step. PRs merge directly to `main` once CI (the sanity check) passes.

## Working philosophy (inherited from Pacific Shift Labs' standing rules)

- Boring and simple first.
- Halt and log, don't self-heal — if something's ambiguous or a spec gap is found, stop and report it in the PR/issue rather than guessing silently.
- Automation presents context, it doesn't make product decisions on Stan's behalf.
- One issue worked at a time in this repo — never dispatch multiple concurrently.
- Self-merge is authorized by default once CI is green — PR code review is a post-merge job, not a pre-merge gate.
