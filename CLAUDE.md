# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file web app for prospecting local businesses in Vallecas, Madrid. No build step, no dependencies, no framework — everything lives in `index.html`.

## Running the app

Open `index.html` directly in a browser. No server needed.

For clipboard API to work without the fallback modal, serve over HTTPS or localhost:

```
npx serve .
```

## Architecture

Everything is in one file (`index.html`): inline CSS, inline JS, HTML structure.

**Data layer** — `localStorage` key `registrador_negocios` stores a JSON array of business records. Shape:

```js
{ id, nombre, categoria, telefono, direccion, tieneWeb, estado, notas, creadoEn }
```

**State** — two mutable variables: `records` (the full array) and `filter` (active status filter string). No framework, no reactivity — DOM is mutated directly.

**Render pattern** — `renderList()` rebuilds the entire list innerHTML from `records` + `filter`. Metrics are the exception: `updateMetrics()` only touches the 4 counter elements (no list re-render). Always call both after any data mutation.

**Status change** — inline `<select>` on each card. On change: update record, `save()`, `updateMetrics()`, update the select's `data-status` attribute (drives CSS color), remove card from DOM if it no longer matches the active filter.

**Copy flow** — `navigator.clipboard.writeText()` with a `<textarea>` fallback modal for `file://` contexts where clipboard is blocked.

## Key rules

- `font-size: 16px` on all form inputs — prevents iOS auto-zoom.
- Status `<select>` color is driven by the `data-status` attribute, not a class. Update it after every status change.
- After adding a business, always reset `filter` to `'Todos'` and sync the filter `<select>` element, so the new record is visible.
- Delete requires the custom confirm modal — never `window.confirm()`.
- "Sin web" metric counts only actionable statuses (`Por contactar`, `Contactado`, `Interesado`), not `Cerrado`/`Rechazado`.
- Per-card WhatsApp button only renders when `tieneWeb === false`.

## Skill routing

When the user's request matches an available skill, invoke it via the Skill tool.

- Product ideas/brainstorming → `/office-hours`
- Architecture → `/plan-eng-review`
- Bugs/errors → `/investigate`
- Visual polish → `/design-review`
- Code review → `/review`
