# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

- The app is a single self-contained file, `index.html` (World Weather dashboard). There is no build step, package manager, linter, or test suite. To run it, open the file in a browser (or serve the folder statically); it needs network access to the Open-Meteo APIs.
- `sample-project/` is only a placeholder with a README; it is unrelated to the app.
- Git remote is `origin` (`AkshyaPadhy/Claude-AI`), branch `main`.

## Architecture of `index.html`

Everything (CSS, markup, JS) lives in one file with no dependencies. The JS flow:

- **Data source:** Open-Meteo forecast API (batched, up to 50 cities per request, `timezone=auto`) and Open-Meteo geocoding API for the city search box. No API keys.
- **State:** a single `state` object (`custom` cities, `weather` Map, `unit`, `continent` filter). Weather is keyed by `key(city)` = `"lat,lon"` rounded to 2 decimals, so lookups depend on that exact format. `DEFAULT_CITIES` is a hard-coded list; user-added cities go in `state.custom` and persist in `localStorage` (`customCities`, `unit`), always wrapped in try/catch.
- **Rendering:** `renderGrid()` and `renderFilters()` rebuild the DOM from scratch (`innerHTML = ""`) on every refresh, unit toggle, or filter change. Anything attached to cards (animations, per-card timers) is recreated each time. `refresh()` refetches all cities then calls `renderGrid()`; an interval triggers it every 2 minutes while the tab is visible.
- **Per-city timezone:** the forecast response's `timezone` is stored with each city's weather and drives the live "Local time" row. `tickClocks()` updates elements with `data-tz` every 30s without refetching.
- **Theming (two independent systems):**
  - Page theme: CSS variables on `:root`, switched via `data-theme` on `<html>`. It follows the viewer's local clock (light 06:00–18:00) until the user flips the switch.
  - Card theme: each card gets `.day` or `.night` from the API's `is_day`, overriding the variables locally, so a card can be dark on a light page and vice versa.
- **Weather codes:** the `WMO` table maps Open-Meteo weather codes to `[text, dayIcon, nightIcon]`; unknown codes fall back to a thermometer.
