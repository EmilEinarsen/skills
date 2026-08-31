# Coding standards

Reviewers read this file. Implementers follow the pointers in `AGENTS.md`.

## Tests

Unit-test a helper that already exists as its own module (date grouping, SQL
compile, schema parse). An inlined route or UI list stays in that file. A new
file whose only caller is a unit test of that list is not a seam.

Each package Vitest config has two projects. `*.test.ts` is `unit` (`TZ=UTC`).
`*.tz.test.ts` is `tz` (`TZ=America/Chicago`). The suffix is the opt-in: that
file runs again in a second zone. Use it when the test ages a timestamp, pins
`now`, builds local date parts, or calls `wallClock`. One suite per helper —
the Chicago run is the second zone, not a second file.

Effect programs keep the worker flow intact —
[`.cursor/rules/effect-usage.mdc`](.cursor/rules/effect-usage.mdc) § Testing.

## Presentation loaders

Query cache or Electric hydrate. The contract is
[`.cursor/rules/presentation-loaders.mdc`](.cursor/rules/presentation-loaders.mdc).

## House style

The convention is the nearest call site in this tree, plus the lint config.
A form neighboring call sites do not use is a miss.

No `switch`. A branch on a known set of keys is a lookup table
(`as const satisfies Record<Key, …>`); a branch on a few conditions is `if`.

Keep an exported abstraction only when removing it makes callers duplicate a
rule, know a representation or integration detail, or repeat policy, invariant,
or error translation. A binding that only gives an existing value or operation
a second name is a Middle Man; call the target directly. Call-site count does
not provide warrant.

## Cleanup

`/cleanup` applies this section after its portable pass.

Use truthiness when only presence or emptiness matters. Use `!!` when the
result must be Boolean. Add a comparison only when its boundary has domain
meaning.

Use `undefined` for absence when the contract permits either nullish value.
Keep `null` only when a domain or external contract distinguishes it.

Presentation enables React Compiler (`viteReact({ compiler: true })`). Do not
add or keep `useMemo`, `useCallback`, or `React.memo` only to prevent
re-renders. Keep manual memoization only when identity is part of an
observable contract.

Keep `snapshot.json` only in the latest generated migration under
`packages/core/migrations`. Delete older snapshots.

If this branch introduced a migration and that work is not on `origin/main`,
confirm the branch migration is still the latest after updating from
`origin/main`. If `origin/main` has a newer migration, delete the branch
migration, regenerate with `pnpm --filter @btr/core db:generate`, and keep
only the newest snapshot.

## File layout

A module with one primary export puts that export first. Helpers, local
types, and constants follow in the order the primary body first uses them,
depth-first.

`buildX` uses `a` then `b`. `a` uses `aHelper`. Order:

`buildX` → `a` → `aHelper` → type `a` needs → `b`

Bindings the primary initializer reads (`validateSearch` schema, default
objects) sit immediately above it. Function bodies may sit below; those
bindings may not.

A file of peer exports (a handler catalog, a types module) groups each
topic the same way: the topic's public binding, then what it uses.
