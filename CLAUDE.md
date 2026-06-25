# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A minimal example/template for building an [Alt1 Toolkit](https://runeapps.org/alt1) app (a browser overlay tool for RuneScape) using webpack and TypeScript. It is intentionally small — one entry point, one image asset, and the bare minimum to demonstrate Alt1's pixel-capture and image-matching APIs.

## Commands

```sh
npm i           # install dependencies (no package-lock.json is committed)
npm run build   # one-off webpack build -> dist/
npm run watch   # webpack --watch, rebuilds on source changes
```

There are no lint or test scripts configured in `package.json`. To try the app, open `dist/index.html` directly in a browser (paste-image functionality works standalone), or load it inside the Alt1 browser and use the "add app" button to get full capture/overlay functionality.

## Architecture

- `src/index.ts` is the single entry point (`webpack.config.js` `entry.main`). Webpack bundles it to `dist/main.js` as a UMD library named `TestApp`, so exported functions (e.g. `capture()`) are reachable from inline HTML handlers via `window.TestApp.capture()` — see the `onclick` handler in `src/index.ts`.
- `src/index.ts` imports `./index.html`, `./appconfig.json`, and `./icon.png` purely so webpack's asset rules copy them into `dist/` alongside the JS bundle; webpack is configured (`webpack.config.js`) to treat `.html`/`.json` and image extensions as `asset/resource` and emit them with their original filename.
- `*.data.png` files (e.g. `src/homebutton.data.png`) are raw image assets loaded through `alt1/imagedata-loader` (configured in `webpack.config.js`), exposed via `a1lib.webpackImages(...)`. This loader strips PNG color-profile metadata that would otherwise shift colors slightly when decoded by the browser — required because Alt1's pixel matching needs exact colors. Load these async results via `imgs.promise` before using them if needed.
- Runtime detection pattern: code checks `window.alt1` to decide whether it's running inside the Alt1 client (full capture/overlay access) or a plain browser (paste-image-only fallback via `a1lib.PasteInput.listen`). Follow this same guard when adding new capture-dependent features.
- `appconfig.json` is Alt1's app manifest (name, icon, permissions, default window size). `alt1.identifyAppUrl("./appconfig.json")` registers the app with the Alt1 client at runtime; outside Alt1, an `alt1://addapp/...` link is shown instead.
- `webpack.config.js` externalizes `sharp`, `canvas`, and `electron/common` — these are optional native deps that `alt1` lib code may use when running under Electron/Node, and must not be bundled for the browser build.
- `tsconfig.json` uses `moduleResolution: "bundler"` and `module: "ESNext"` specifically to match webpack's resolution behavior (e.g. `resolvePackageJsonExports`) — don't change these without checking that `ts-loader`/webpack still resolve the `alt1` package correctly.
