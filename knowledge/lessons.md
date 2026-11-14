# Lessons

## Our published types include location-conflation's

`lib/types.ts` uses `LocationSet` from `@rapideditor/location-conflation`, so `dist/ts/` imports from it, and its `.d.ts` files load for every consumer. If they're broken, our types are broken too. location-conflation 3.0.1 used the global `GeoJSON` namespace and failed for TypeScript 7 consumers, even though publint and arethetypeswrong both passed for us. Only a scratch project with the packed tarball installed (`"types": []`, `"skipLibCheck": false`) caught it. Repeat that check after upgrading location-conflation.

## TypeScript patterns

- Inside a `declare module '…'` block in `lib/ambient.d.ts`, the global `GeoJSON` namespace isn't reachable. Use `import type { … } from 'geojson'` inside the block.
- Add `as const` to a literal array you loop over (`for (const prop of ['name', 'description'] as const)`), so each item narrows to a literal key instead of `string`.
- To test invalid input from untyped callers (e.g. `simplify(null)`), mark the line with `// @ts-expect-error -- reason` rather than casting.
