# memory-map-mind-elixir Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-12-07

## Active Technologies
- Vanilla JavaScript (ES2020), HTML5, CSS3 — no TypeScript, no build step + None — standard browser APIs only (001-coer-form)
- Shared JSON project file via File System Access API (with `<input type="file">` fallback) (001-coer-form)
- TypeScript 5.3, HTML5 (static launcher + COER) + Vite 7.3.1, React 18.2, mind-elixir ^5.3.7 (001-suite-launcher)
- N/A (launcher is navigation-only; no data persistence) (001-suite-launcher)
- HTML5 + vanilla JavaScript (ES6), inline CSS + None — native browser APIs only (HTML5 pointer events for drag-and-drop, File API for save/load) (001-tmm)
- File-based JSON save/load via browser File API; shared `effectivenessToolkit` envelope (001-tmm)

- TypeScript 5.x (ES2020 target) in-browser SPA built with Vite 5 + mind-elixir-core (fork), lightweight utility helpers only (built-in Blob/File APIs for download; no network deps) (001-plan-memory-map)

## Project Structure

```text
backend/
frontend/
tests/
```

## Commands

npm test && npm run lint

## Code Style

TypeScript 5.x (ES2020 target) in-browser SPA built with Vite 5: Follow standard conventions

## Recent Changes
- 001-tmm: Added HTML5 + vanilla JavaScript (ES6), inline CSS + None — native browser APIs only (HTML5 pointer events for drag-and-drop, File API for save/load)
- 001-suite-launcher: Added TypeScript 5.3, HTML5 (static launcher + COER) + Vite 7.3.1, React 18.2, mind-elixir ^5.3.7
- 001-coer-form: Added Vanilla JavaScript (ES2020), HTML5, CSS3 — no TypeScript, no build step + None — standard browser APIs only


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
