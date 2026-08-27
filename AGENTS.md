## What this is

A standalone, single-file HTML tool: a Gantt chart for product roadmaps, built with Plotly.js loaded from CDN. No build step, no bundler, no npm dependencies. The entire product is one file — `index.html` — that opens directly in a browser.

## Product boundary

- **Program Increments (PI) and Sprints are overlaid as bands at the top of the chart.** Task activity bars render below them, sharing the same date-based x-axis, so PI/sprint boundaries align visually with the tasks underneath.
- **Milestones are not a separate entity.** A milestone is a `TASKS` entry where `start === end` (a same-day task) — it has a lane, status, owner, and description exactly like any other task, created through the same Task form. The only difference is visual: it renders as a diamond marker instead of a bar, since a zero-width bar isn't visible or clickable.
- **Tasks are organized into named Lanes (swimlanes), not one row per task.** All tasks assigned to the same lane (including same-day "milestone" tasks) render on that lane's single row. Lanes are independently manageable (add/edit/delete/reorder), separate from tasks themselves.
- **Task color is chosen manually, per task — not derived from status.** The original status-driven legend/coloring from #6 was removed as unhelpful in practice. Each task has its own `color` field, set via a color picker in the Task form. `status` still exists as a field (visible via hover and the click-to-overlay dialog) but no longer drives any visual color or legend.
- **Tasks carry a longer freeform description, with three separate ways to see it.** (1) The hover tooltip — quick-glance summary (name/date(s)/status/owner). (2) Click-to-overlay dialog with the full description. (1) and (2) are both screen-only and disappear on PNG export. (3) **`descriptionPosition`** (`'hidden' | 'above' | 'below'`, default `'hidden'`) bakes the description as a static, always-visible chart annotation — the only one of the three that survives a PNG export.
- **Chart title and x-axis label are editable through the Configure screen, not just hand-edited constants.** Every other piece of roadmap content (PIs, Sprints, Lanes, Tasks) is edited via Configure; title/axis-label followed the same pattern once that inconsistency was noticed (2026-08-26). Both persist through Open/Save like the rest of the roadmap data, with safe fallback defaults (`"Product Roadmap"` / `"Date"`) for older saved files missing these keys.
- **This is a personal planning tool, not a hosted service.** There is no backend, no deploy target, no container. The deliverable is the file itself.
- **This is a real roadmap-authoring tool, not just a static viewer.** The product owner creates, edits, and plans roadmaps directly in the tool — adding/editing/removing Program Increments, Sprints, Lanes, and Tasks (including milestones, as same-day tasks) through an in-app UI — to produce a clean visual to brief to leadership. The tool started as "hand-edit the JS arrays," and has grown into "open a roadmap file, edit it in the app, save it back," and now "configure chart-level settings in the app too."
- **Two distinct view modes: Chart and Configure.** A toggle at the top of the page shows either the chart (plus PNG export) or the editing/configuration UI (PI/Sprint/Lane/Task forms and lists, Chart Settings, plus Open/Save) — never both at once.
- **Exportable to PNG** for dropping directly into briefs/slide decks (see #13). The chart uses the full available viewport width (no fixed max-width) and its height scales with lane count.

## Architecture

- Single file: `index.html`. HTML, CSS, and JS all inline.
- Plotly.js via CDN script tag, pinned to a specific version (`2.35.2` as of this writing) — don't float on `latest`.
- Data collections — same shapes whether they come from the built-in sample data or a loaded file:
  - `PROGRAM_INCREMENTS`: `[{ name, start, end }]`
  - `SPRINTS`: `[{ name, start, end, piName }]` (piName links a sprint to its parent PI for validation/grouping, not required for rendering)
  - `LANES`: `[{ name }]` — display order in this array is the display order in the chart (top to bottom). No separate `order` field; reordering means reordering this array.
  - `TASKS`: `[{ name, start, end, status, owner?, lane, description?, color?, descriptionPosition? }]` — `lane` references a `LANES` entry's `name`, same referencing convention as `SPRINTS.piName`. `color` is an optional hex string set manually per task (not derived from status). `descriptionPosition` is `'hidden' | 'above' | 'below'`, default `'hidden'` — when not hidden, bakes `description` as a static chart annotation so it survives PNG export. **A task where `start === end` is a milestone** — same schema, no extra field needed, only the rendering shape differs.
  - Chart-level settings: `CHART_TITLE` (default `"Product Roadmap"`), `X_AXIS_TITLE` (default `"Date"`), and `CHART_THEME` (default `"vizimap"`) — plain `let` variables, editable via Configure's "Chart Settings" section, not `const`s baked into the render function. The custom Vizimap theme uses the logo palette and presentation styling; the ten built-in Plotly themes remain available.
- Gantt bars use the standard Plotly technique: `type: 'bar', orientation: 'h'`, date-typed x-axis, `base` = start date, `x` = duration. A task with `start === end` (a milestone) renders instead as a scatter-trace point with `marker.symbol: 'diamond'`, positioned in its lane's row like any other task in that lane.
- PI and Sprint bands render as their own rows at the top (same technique), with vertical dashed gridlines at PI/sprint boundaries extending down through the task rows below, so alignment is a literal visual gridline, not just proximity.
- **Task rows are one-per-lane, not one-per-task.** Multiple tasks (bars and/or milestone diamonds) assigned to the same lane share that lane's row. **Deliberate scope boundary:** if two tasks in the same lane have overlapping date ranges, they will visually overlap on the chart — this is accepted as useful signal (a real scheduling conflict), not a bug. Automatic sub-row packing/offsetting for overlapping same-lane tasks is explicitly out of scope unless revisited later.
- **No legend.** Removed along with status-driven coloring — it was static and unhelpful. Task color is per-task and manual (see above); don't reintroduce a legend without a specific reason to.
- **Click vs. hover vs. inline annotation — three distinct ways to see a task's description, each for a different purpose:** hover (native Plotly tooltip) = quick glance on-screen only. Click = deliberate, on-screen dialog with the full description, dismissible. `descriptionPosition` = always-visible, baked into the chart itself, the only one of the three that survives a PNG export. All three can coexist on the same task; none replaces the others.

## Persistence (decided 2026-08-26)

- **Mechanism: the File System Access API** (`window.showOpenFilePicker` / `showSaveFilePicker`). "Open Roadmap" picks a JSON file from disk; edits happen in memory; "Save" writes back to that same open file handle — no server, no download-then-manually-replace-the-file step. This is the intended day-to-day workflow: the JSON file sits next to `index.html` like a companion data file, opened and saved in place.
- **Chromium-only by design, not an oversight.** The File System Access API only works in Chrome/Edge, not Firefox or Safari. Confirmed: Stan only uses Chrome/Edge, so **no fallback for other browsers is required.** Don't build one. If the API is unavailable, a clear error message is enough.
- **First-run behavior:** if no file has been opened yet, the app falls back to the built-in sample data already in the file — so `index.html` still opens and shows something meaningful with zero setup, same as today.
- **JSON schema on disk** is a single object wrapping the data collections plus chart-level settings: `{ programIncrements: [...], sprints: [...], lanes: [...], tasks: [...], chartTitle: "...", xAxisTitle: "...", chartTheme: "..." }`. Milestones need no schema entry of their own — they're just `tasks` entries. **Any key missing from an older saved file must fall back to its built-in default, not error** — this applies to chart-level settings added later than the rest of the schema and to any future schema additions.

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
