# Vizimap

A standalone, single-file HTML Gantt chart for product roadmaps — built to plan and present Program Increments, Sprints, and task activity in one clean view, and export it straight into a leadership brief.

No install, no build step, no backend. Open `index.html` in a browser and it works.

## What makes it different from a normal Gantt chart

Program Increment and Sprint timelines are overlaid as bands at the top of the chart, with dashed vertical gridlines marking their boundaries straight down through the task rows below — so it's immediately visible which sprint (and which PI) any given task falls into, not just its raw calendar dates.

## Status

🚧 Actively being built. See the [tracking issue](https://github.com/m3rrym4n/vizimap/issues/8) for current progress and the full build order.

**Built:**
- [x] Core data model and rendering scaffold

**In progress / planned:**
- [ ] Program Increment band
- [ ] Sprint band
- [ ] PI/Sprint alignment gridlines
- [ ] Task activity Gantt rows
- [ ] Status color-coding and legend
- [ ] Date range slider and "today" marker
- [ ] Configurable x-axis (Calendar / Sprint / PI tick modes)
- [ ] Export to PNG for leadership briefs
- [ ] Open/save roadmaps as a JSON file (in place, via the File System Access API)
- [ ] In-app editing: add/edit/delete Program Increments, Sprints, and Tasks

## Using it

Open `index.html` in **Chrome or Edge**. (This is a deliberate choice, not a gap — roadmap save/load uses the File System Access API, which only Chromium browsers support.)

Until in-app editing lands, roadmap data lives as plain arrays near the top of `index.html`'s `<script>` block — edit them directly in a text editor:

```js
const PROGRAM_INCREMENTS = [
  { name: "PI-1", start: "2025-01-06", end: "2025-03-28" },
];

const SPRINTS = [
  { name: "Sprint 1.1", start: "2025-01-06", end: "2025-01-24", piName: "PI-1" },
];

const TASKS = [
  { name: "Define roadmap", start: "2025-01-06", end: "2025-01-10", status: "Done", owner: "Stan" },
];
```

Once roadmap persistence lands, the same shapes will live in a `.json` file you open and save directly from the app — no code editing required at that point.

## Architecture

- One file: `index.html`. HTML, CSS, and JS all inline.
- [Plotly.js](https://plotly.com/javascript/) loaded from a CDN, pinned to a specific version — no npm, no bundler, no other dependencies.
- Gantt bars, PI bands, and Sprint bands all use the same technique: horizontal bars on a date-typed axis.

## Contributing

This repo is built issue-by-issue by an autonomous coding agent (Codex, dispatched via [Variflex](https://github.com/m3rrym4n/variflex)) against a fixed set of standing rules. See [`AGENTS.md`](./AGENTS.md) for the full architecture, scope, and working philosophy this repo is held to.
