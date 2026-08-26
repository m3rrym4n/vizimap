# <img width="2172" height="724" alt="vizimap-logo" src="vizimap-logo.png" />


A standalone, single-file HTML Gantt chart for product roadmaps — built to plan and present Program Increments, Sprints, and task activity in one clean view, and export it straight into a leadership brief.

No install, no build step, no backend. Open `index.html` in a browser and it works.

## What makes it different from a normal Gantt chart

Program Increment and Sprint timelines are overlaid as bands at the top of the chart, with dashed vertical gridlines marking their boundaries straight down through the task rows below — so it's immediately visible which sprint (and which PI) any given task falls into, not just its raw calendar dates. Tasks are grouped into named Lanes (swimlanes), and a same-day task automatically renders as a diamond milestone marker instead of a bar.

## Status

🚧 Actively being built. See the [tracking issue](https://github.com/m3rrym4n/vizimap/issues/8) for current progress and the full build order.

**Built:**
- [x] Core data model and rendering scaffold
- [x] Program Increment band
- [x] Sprint band
- [x] PI/Sprint alignment gridlines
- [x] Task activity Gantt rows
- [x] Status color-coding and legend
- [x] Date range slider and "today" marker
- [x] Configurable x-axis (Calendar / Sprint / PI tick modes)
- [x] Export to PNG for leadership briefs
- [x] Open/save roadmaps as a JSON file (in place, via the File System Access API)
- [x] In-app editing: add/edit/delete Program Increments, Sprints, and Tasks
- [x] Lanes (swimlanes): tasks grouped into named, reorderable rows instead of one row per task
- [x] Milestones: a same-day task automatically renders as a diamond marker

**In progress / planned:**
- [ ] Chart / Configure view toggle (#32)
- [ ] Click-to-overlay task description on the chart (#33)
- [ ] Visual design pass — modern styling matching the logo's palette (#23, deferred until the above lands)

## Using it

Open `index.html` in **Chrome or Edge**. (This is a deliberate choice, not a gap — roadmap save/load uses the File System Access API, which only Chromium browsers support.)

Roadmap data (Program Increments, Sprints, Lanes, Tasks) can be opened and saved directly as a `.json` file from within the app — no code editing required. Until you've opened a file of your own, the app shows built-in sample data so `index.html` is useful with zero setup.

## Architecture

- One file: `index.html`. HTML, CSS, and JS all inline.
- [Plotly.js](https://plotly.com/javascript/) loaded from a CDN, pinned to a specific version — no npm, no bundler, no other dependencies.
- Gantt bars, PI bands, and Sprint bands all use the same technique: horizontal bars on a date-typed axis. Milestones (same-day tasks) render as diamond-symbol markers on the same axis instead.

## Contributing

This repo is built issue-by-issue by an autonomous coding agent (Codex, dispatched via [Variflex](https://github.com/m3rrym4n/variflex)) against a fixed set of standing rules. See [`AGENTS.md`](./AGENTS.md) for the full architecture, scope, and working philosophy this repo is held to.
