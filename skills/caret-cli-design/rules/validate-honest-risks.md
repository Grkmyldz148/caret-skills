# validate-honest-risks

> Run last, after the other two validators. Catches the "plan
> reads great until the user starts the migration and finds out
> what was glossed over" failure mode.

The risks section in your plan must include every actual risk you
found in the inventory + mapping. Don't soften, don't omit, don't
package losses as gains. The plan is for an adult.

## The check

Walk back through:

1. The inventory — did you see any `\x07` / bell? Any inline
   hex colours? Any emoji-as-state-marker? Any pre-formatted
   ANSI strings?
2. The mapping — any "NO FIT" rows? Any cosmetic shifts the
   user will notice (spinner cadence, glyph choice)?
3. The adoption order — any step the user might find
   surprising in scope (e.g. "this touches 50 files")?

For each, ask: is this risk in the risks section?

If not, add it. The risks section is short — it can carry the
weight.

## Honesty patterns

These are the failure modes this validator catches.

### Soft-pedalling a manifesto blocker

✗ **Bad:** "Caret has a different approach to error
notifications."

✓ **Good:** "Caret bans the terminal bell. `deploy.ts:67`
currently rings the bell on error; that goes away. If this is
non-negotiable, Caret won't fit."

### Burying the dependency removal in optional work

✗ **Bad:** "After the migration, `chalk` and `ora` can be
removed if you want."

✓ **Good:** "When the migration completes, `chalk`, `ora`,
`inquirer`, `figures`, `listr` are no longer imported
anywhere. Removing them is a one-line `pnpm remove`. Doing so
is recommended (keeps deps tight) but optional."

The first version sounds like the user can ship the migration
half-applied. The second version makes the trailing cleanup
explicit and finite.

### Hiding test-snapshot churn

✗ **Bad:** (no mention of tests)

✓ **Good:** "11 snapshot tests reference current CLI output.
After migration, all 11 need re-recording. Budget ~30 min for
this; the snapshots themselves are mechanical to update."

If the inventory included test files, the risks section must
mention them. Surprise test failures are the #1 reason
migrations get reverted.

### Vague effort estimates

✗ **Bad:** "This is a small migration."

✓ **Good:** "Adoption touches 8 files (`src/index.ts`,
`src/commands/{init,deploy,configure,info}.ts`,
`src/lib/log.ts`, `src/lib/print.ts`, `bin/cli.js`). Estimated
2-3 hours for a developer familiar with the codebase."

Numbers > adjectives. The user can plan around numbers.

## What "honest" means in the risks section

Three rules:

1. **Every risk has a name and a location.** "Terminal bell at
   `deploy.ts:67`" beats "the CLI uses the bell somewhere."
2. **Every risk has a severity.** `manifesto-blocker` /
   `cosmetic` / `low` / `medium` / `high`. Calibrated.
3. **Every risk has an out.** "If this is unacceptable, the
   alternative is X." For some risks the out is "keep the
   existing CLI" — that's a legitimate out, write it.

## What to record

Pass / fail. If pass: "validate-honest-risks: ✓". If fail,
update the risks section. Then ship the plan.

After all three validators pass, the plan is ready to deliver
to the user. Stop authoring; deliver.
