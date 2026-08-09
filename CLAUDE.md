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
4. **Render functions** (`renderTabs`, `renderSchedule`, `renderSpotDetail`, `renderFood`, `renderModelCourse`, `renderTodoList`, `renderWeekOverview`) — each does a full innerHTML re-render of its container from current state (`activeDay` / `days` / `spots`). There's no diffing; switching days or changing a spot's dropdown just calls `renderAllForActiveDay()` again, which also refreshes the week overview.
5. **`nearbySubSpots`** — keyed by each spot's `nearbyCluster` (e.g. `tokorozawa`, `kasukabe`), lists nearby detour spots (food, shopping malls, roadside stations/道の駅, etc.) shown in a filterable "周辺のサブスポット" panel under each spot's detail.
6. **週間予定一覧 (`renderWeekOverview` / `#weekOverviewGrid`)** — a PC-oriented grid showing all 7 days at once (1 col on mobile, up to 4 cols on wide screens via a CSS "breakout" from the page's normal `max-w-2xl` reading column). Each card has its own destination `<select>` (reuses `buildSpotSelect`) and a swap-with-another-day `<select>` (`buildSwapControl`), so the whole week can be rearranged without switching day tabs — meant for viewing/editing together as a family on a larger screen. `printWeekBtn` toggles a `body.print-week-mode` class to print just this grid instead of the single active day.
7. **Weather (`loadAllWeather` / `weatherByMunicipality`)** — fetches a 16-day forecast per municipality (hardcoded lat/lon in `WEATHER_COORDS`, derived from each spot's `area` via `getMunicipality`) from the free, key-less Open-Meteo API (`api.open-meteo.com`) on page load, client-side, no backend. Failures are swallowed silently (weather badges just don't appear) so the app still works offline/without network. Shown inline in both the week overview cards and each spot's detail (`buildSpotInfoBlock`).
8. **Spot tags (`getSpotTags`)** — regex-derived badges (🐻 bear-sighting area / 💦 water play / 🏠 indoor-cool / 🥵 outdoor-heat / 📅 reservation recommended) computed from each spot's existing `description`/`highlights`/`tips`/`recommendations.reservation` text and `bearAlert` field — not a separately maintained data field, so keep those source fields accurate rather than editing tags directly.

To add or edit a destination: add/edit an entry in `spots` (include an `area` that starts with a real municipality name, e.g. `"所沢市 東所沢"`, so `getMunicipality`/weather lookups work — add the municipality to `WEATHER_COORDS` if it's a new one), drop its images in `images/` following the `<id>-1.jpg`/`<id>-2.jpg`/`<id>-3.jpg` (and optionally `<id>-food-1.jpg`/`<id>-food-2.jpg`) naming convention, then reference it from `days` (or leave it selectable-only via the dropdown).

Known gap: several `foods[].image` entries reference files that don't yet exist in `images/` (e.g. `forest-food-*.jpg`, `hashidate-food-*.jpg`, `saiboku-food-*.jpg`, `shinrinkoen-food-*.jpg`, `gaikaku-food-2.jpg`) — these are placeholder rows meant to be replaced with real restaurant picks and photos before the trip.
