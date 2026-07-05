# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page trip-planning app (in Japanese) for a family trip to Saitama, Japan (お盆 8/11–8/16 2026, plus one spare day). It's a static site: `index.html` contains all markup, Tailwind (via CDN `<script>`), and vanilla JS in one inline `<script>` block — there is no build step, no package manager, and no test suite.

## Running it

There is no dev server or build command. Open `index.html` directly in a browser, or serve the directory with any static file server (e.g. `python3 -m http.server`) from *this* directory.

Note: image `src` paths are hardcoded as absolute (`const imgBase = "/Plan_Guide_2026/images"`), matching the GitHub Pages project-page path (`https://<user>.github.io/Plan_Guide_2026/`). If you serve the site from a different base path (e.g. `file://` directly, or a root-mounted local server), images will 404 — either adjust `imgBase` locally or serve from a path ending in `/Plan_Guide_2026/`.

## Architecture

Everything lives in `index.html`, structured as:

1. **`spots` array** — one object per destination (id, name, area, images, description, highlights, tips, transport options by car/train, fees/reservation links, a Google Maps URL, example restaurant entries, an hour-by-hour `modelCourse`, and `recommendations` for heat/traffic/kids/rain/reservation/packing). This is the single source of truth for all spot content.
2. **`days` array** — maps each of the 7 days (Day 1–6 plus a 予備日/spare day) to a `spotId`. The spot assigned to a day can be changed at runtime via a `<select>` dropdown, which mutates `days[i].spotId` directly (no framework/state store).
3. **`todoItems` array** — static checklist content; check state persists to `localStorage` under key `saitama-trip-todo`, keyed by each item's `id`.
4. **Render functions** (`renderTabs`, `renderSchedule`, `renderSpotDetail`, `renderFood`, `renderModelCourse`, `renderTodoList`) — each does a full innerHTML re-render of its container from current state (`activeDay` / `days` / `spots`). There's no diffing; switching days or changing a spot's dropdown just calls `renderAllForActiveDay()` again.

To add or edit a destination: add/edit an entry in `spots`, drop its images in `images/` following the `<id>-1.jpg`/`<id>-2.jpg`/`<id>-3.jpg` (and optionally `<id>-food-1.jpg`/`<id>-food-2.jpg`) naming convention, then reference it from `days` (or leave it selectable-only via the dropdown).

Known gap: several `foods[].image` entries reference files that don't yet exist in `images/` (e.g. `forest-food-*.jpg`, `hashidate-food-*.jpg`, `saiboku-food-*.jpg`, `shinrinkoen-food-*.jpg`, `gaikaku-food-2.jpg`) — these are placeholder rows meant to be replaced with real restaurant picks and photos before the trip.
