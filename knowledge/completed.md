# Completed

## November 2026

- **ESLint → oxlint** ([#883](https://github.com/osmlab/osm-community-index/pull/883)): same rules, much faster `check:lint`, and it cleared the only `bun audit` advisory. Unblocked TypeScript 7.
- **TypeScript 7** ([#884](https://github.com/osmlab/osm-community-index/pull/884)): tests are now TypeScript and type-checked, `isolatedDeclarations` is on, and the casts and hidden `any` values in `lib/` and `scripts/` are gone (see `decisions.md`). The build now gives a clear error for a FeatureCollection with more than one feature, instead of crashing.
- **ESM only and package checks** ([#885](https://github.com/osmlab/osm-community-index/pull/885)): dropped the CommonJS build, added publint and arethetypeswrong to `bun run all`, and upgraded to location-conflation 4, which fixed our published types for TypeScript 7 consumers. Needs a major release (7.0.0).
- **Agent setup**: Copilot prompts became Agent Skills in `.claude/skills/`, and the scaffold sync added this `knowledge/` folder.

## April 2026

- **Strict TypeScript** across `src/`, `lib/` and `scripts/`, modeled after name-suggestion-index [#12067](https://github.com/osmlab/name-suggestion-index/pull/12067).
