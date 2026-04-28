# Animation Demo

Small Vite app that chains three motion vignettes: a loader, a staggered text reveal, and a Matter.js pendulum that starts when the canvas scrolls into view.

## Requirements

- Node.js with npm

## Setup and local run

```bash
npm install
npm run dev
```

- **dev**: Vite dev server (default `http://localhost:5173`).
- **build**: Production bundle in `dist/`.
- **preview**: Serve the production build locally.

## Deploy (GitHub Pages)

The app is configured for a project site under `/animation-demo/`:

```4:5:vite.config.js
export default defineConfig({
    base: '/animation-demo/',
```

```bash
npm run build
npm run deploy
```

`deploy` runs `gh-pages -d dist`. If you fork or change the repo name, update `base` in `vite.config.js` to match the GitHub Pages URL path, then rebuild.

## Architecture

| Piece | Role |
|--------|------|
| `index.html` | Mounts `#app` and loads `src/main.script.js`. |
| `src/main.script.js` | Runs scenes in order: loader (await) → message → rigid bodies. |
| `src/modules/utils.js` | `addTo(html, selector)` and `remove(selector)` for DOM helpers used by each module. |
| `src/modules/loader/` | Builds loader markup, runs Motion `animate` sequences (spring, stagger), removes `.loader` when done. |
| `src/modules/message/` | Wraps each character in spans, animates opacity with stagger. |
| `src/modules/rigidBodies/` | Adds `.frame`, creates Matter engine/render/pendulum/trail, starts the runner only when `.frame` is sufficiently in view (`motion` `inView`). |

Dependencies: **motion** (animations, scroll/in-view), **matter-js** (physics in the rigid-body scene).

## Constraints and pitfalls

- **`#app` must exist** in `index.html`; modules append to it via `utils.addTo`.
- **`utils.addTo` throws** if the selector matches no element (no optional chaining on `insertAdjacentHTML`).
- **Rigid-body scene**: `buildEngine('.frame')` reads `element.offsetWidth` / `offsetHeight` after the node is in the DOM; very narrow containers may affect pendulum placement.
- **Path-sensitive assets**: With `base: '/animation-demo/'`, absolute asset paths in HTML should stay consistent with Vite’s handling (favicon uses `/cat.svg` from `public/`).

## Extending

Add a new module under `src/modules/`, import its default in `main.script.js`, and follow the pattern: `buildHTML` → `utils.addTo(..., '#app')` → run animations or side effects → remove or leave DOM as needed.
