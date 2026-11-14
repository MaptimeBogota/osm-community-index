# Decisions

## TypeScript setup

- `tsconfig.json` type-checks everything (`src/`, `lib/`, `scripts/`, `test/`) with `noEmit`. `tsconfig.build.json` extends it and only emits declarations for the published entry point (`src/oci.ts`) to `dist/ts/`. The JavaScript bundles come from `scripts/build_js.ts`, not `tsc`.
- On top of `strict`, it turns on `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noPropertyAccessFromIndexSignature` and the `noUnused*` checks. None of these are part of `strict`.
- `declaration` + `isolatedDeclarations` keep the published API from changing by accident: exported functions need explicit types.
- `"types": ["bun"]` is required: TypeScript 7 doesn't load `@types/*` automatically, and four scripts use the `Bun` global without importing from `'bun'`.
- Tests are TypeScript so they're type-checked too. Before November 2026 they were `.js` and `tsc` skipped them.
- `lib/ambient.d.ts` declares the untyped dependencies (`@mapbox/geojson-area`, `@mapbox/geojson-rewind`, `geojson-bounds`).

## GeoJSON types are imported, not global (November 2026)

Use `import type * as GeoJSON from 'geojson'`, not the global `GeoJSON` namespace that `@types/geojson` declares. TypeScript 7 doesn't load `@types/*` automatically, so a published `.d.ts` that uses the global fails for consumers with "Cannot find namespace 'GeoJSON'". Our own `.d.ts` files don't use it, but `scripts/build_json.ts` used to rely on it, and it only resolved because `lib/ambient.d.ts` happened to import `geojson`.

This reverses the April 2026 convention of using the global with no import.

## Parsed data is `unknown` until it's checked (November 2026)

`Bun.file().json()` and `JSON5.parse()` return `any`, which turns off type-checking for everything that touches the result, and the compiler never says so.
- Config-like JSON (`package.json`, `schema/*.json`, `src/defaults.json`) is imported as typed JSON modules instead.
- Feature files are parsed as `unknown` and narrowed with the type guards in `scripts/build_json.ts` (`isFeature`, `isFeatureCollection`) before use.
- Resource files are checked against `schema/resource.json` right after parsing (`validateFile()` is an assertion function), and again after cleanup. Checking first means cleanup can rely on the types, with no defensive fallbacks for missing `locationSet` or `strings`. It also means a property the schema doesn't allow is an error, rather than being silently dropped. Rejected: casting the parsed value to `OciResource`, which hides malformed input.

## ESM only (November 2026, released in 7.0.0)

The package ships only an ES module (`dist/js/oci.mjs`) and the browser IIFE (`dist/js/oci.iife.js`). `require()` loads the ES module, which needs Node 20.19+ or 22.12+. The README says so. There's deliberately no `engines.node`: Yarn 1 fails installs on a mismatch, even for devDependencies. `bun run package` checks the packed tarball with publint and arethetypeswrong (`--profile esm-only`).

## Type-aware linting: not yet (November 2026)

oxlint runs without type information: `.oxlintrc.json` uses the syntax-only `typescript` plugin, which matches what the old ESLint setup did (typescript-eslint `recommended`, not `recommendedTypeChecked`). Type-aware rules (`no-unsafe-*`, `no-floating-promises`, `no-unnecessary-condition`, …) would need the `oxlint-tsgolint` package and `oxlint --type-aware`.

A one-off type-aware run during the TypeScript 7 upgrade found about 40 values typed `any` that came from `Bun.file().json()` and `JSON5.parse()`, three un-awaited promises, and a build crash on multi-feature FeatureCollections. All of these are fixed. On the clean code it costs almost nothing (lint stays under 0.2 s).

Held off for now because:
- It isn't part of the shared agent-practices setup, and adopting it in one project would drift from the others. If we want it, add it there for every project, or as an occasional audit step in `/checkup`.
- It adds a native dependency that has to track the TypeScript version, and oxlint's type-aware support was still a preview when last checked.

Revisit if hidden `any` values creep back in, or once agent-practices adopts it.
