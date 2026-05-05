# validate-no-orphan-mappings

> Run after `validate-runs-after-each-step.md`. Catches the
> "plan covers most of the CLI, ignores the rest" failure mode.

Every entry in your inventory must appear in the mapping table.
Surfaces that show up in the inventory but not the mapping are
orphans — the user will hit them mid-migration, find no plan,
and stall.

## The check

Two lists side by side:

1. The inventory list from `pipeline-inventory.md`.
2. The mapping table from `pipeline-mapping.md`.

For every inventory entry, confirm there's a row in the
mapping table that covers it. The cover can be:

- A direct map (mapping says "init.ts:8 → prompt.text")
- An explicit "NO FIT" with rationale
- A grouping (mapping says "All chalk usage → paint helpers
  per file")

If an inventory entry is silent in the mapping, it's an orphan.

## How to grep for orphans quickly

If your inventory has 30+ entries, scanning manually misses
things. Instead:

```
For each surface in inventory:
  Search the mapping table text for the file:line reference.
  If not found, flag as orphan.
```

Most orphans cluster around:

- **Internal log / debug helpers** that the inventory lists but
  the surface taxonomy doesn't have a category for. Group them
  under "log" surface, not orphans.
- **One-off `console.log` calls in handlers** that don't fit a
  named surface. Either add a specific row to the mapping
  ("`init.ts:23` debug print → `log.debug`") or note as a
  group ("All raw `console.log` calls → `log.info`").
- **Help text formatters** that the inventory caught but
  surface-taxonomy.md flagged as a special case (parser-coupled).
  Make sure the mapping addresses help formatting explicitly,
  even if the recommendation is "keep parser's renderer, theme
  with paint helpers."

## What "covered" means

A mapping row covers an inventory entry if a developer reading
the plan can apply the mapping to that specific file:line and
know what to do. "All logging gets replaced" is a cover for
ten `console.log` calls only if the plan says "replace each
`console.log` with `log.info`" — vague mappings don't cover
specific surfaces.

## Examples

### Orphan

Inventory entry: `src/commands/info.ts:14 — chalk.dim version output`

Mapping table covers prompts, spinners, errors, splash. Doesn't
mention `info.ts:14`. **Orphan.**

Fix: add a row. Likely surface = key-value (the version output
is a `name: value` pair). Mapping: `key-value`.

### Covered

Inventory entry: `src/lib/log.ts:5 — wraps console.log with timestamp`

Mapping row: "All logging → replace with `log` from caret;
`src/lib/log.ts` becomes a re-export of `log.info` etc."

This covers the inventory entry — the developer can apply the
mapping to that file directly.

### Grouped cover (also fine)

Inventory entries:
- `src/index.ts:14 — chalk.bold.cyan banner`
- `src/index.ts:22 — chalk.bold version line`
- `src/index.ts:30 — chalk.dim tagline`

Mapping row: "Splash + banner: replace `index.ts:14-30` with
`<Splash logo={...} version={...} tagline={...} />`."

Covered by a single row referencing the line range.

## What to record

Pass / fail. If pass: "validate-no-orphan-mappings: ✓".
If fail, list each orphan inventory entry. Then go back to
`pipeline-mapping.md`, add the missing rows, and re-run the
validator.
