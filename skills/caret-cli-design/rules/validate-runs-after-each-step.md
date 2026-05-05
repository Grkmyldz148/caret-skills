# validate-runs-after-each-step

> Run before delivering the plan. The single check that catches
> the "this plan is impossible to execute incrementally"
> failure mode.

A plan that requires steps 1, 5, and 7 to be applied together to
compile is not an adoption plan — it's a single-shot rewrite
disguised as a sequence. Catch this before delivery.

## What to verify

For each step in `pipeline-adoption-order.md`'s output, mentally
(or actually) walk the user's repo through:

1. Apply step N's changes.
2. Run `tsc --noEmit` — does the project still compile?
3. Run the CLI's main happy path — does it work?
4. Look at the user-facing output — does it still convey the
   same information (even if styled differently)?

If any answer is "no" for any step, the plan is broken.

## How to mentally walk the steps

You don't have to actually run the user's repo. But you do have
to read each step's "Files touched" list and confirm:

- **No step references a Caret import that step N-K hasn't
  installed yet.** If step 4 imports `paint.success` but the
  install step is step 6, reorder.
- **No step removes a function that another file still calls.**
  If step 3 deletes `src/lib/log.ts` but step 5 still uses
  `console.log` in commands that step 5 hasn't migrated yet,
  the project is fine but the *progression* skips behaviour
  changes.
- **No step depends on a not-yet-shipped Caret component.** If
  the mapping recommends a "command-palette" component and that
  component appears in `pipeline-author-customs.md` (sister
  skill), the adoption step is "create the component first via
  `create-caret-component`, then return."

## The walkthrough table

For every step, fill this in:

| Step | After this step compiles? | After this step the CLI runs? | Output visibly the same? |
| ---- | --- | --- | --- |
| 1 | ✓ | ✓ | ✓ (no visible change) |
| 2 | ✓ | ✓ | ✓ |
| 3 | ✓ | ✓ | logs styled, but information identical |
| 4 | ✓ | ✓ | error blocks styled, behaviour identical |
| ... |

If any cell is ✗, fix the order or split the step.

## Common failures and fixes

### "Step references not-yet-installed component"

Symptom: step 4 imports `<Spinner>` but `npx caret-cli add
spinner` is in step 6.

Fix: move the install command earlier. Often this means
"install all components in step 1 instead of one per step."
That's fine — installation is cheap, the migration of *call
sites* is the work.

### "Step removes a helper that another file still uses"

Symptom: step 3 says "delete `src/lib/log.ts`" but step 5
still has `import log from '../lib/log'`.

Fix: replace contents of `log.ts` (re-export Caret's `log`)
instead of deleting. Delete only after all callers are
migrated.

### "Step requires a refactor that breaks behaviour temporarily"

Symptom: replacing `inquirer` with Caret's `prompt` requires
changing async patterns; mid-migration the prompt resolves
synchronously and the flow breaks.

Fix: migrate one prompt at a time. Each call site is its own
sub-step. The plan can have step "5a Replace init.ts:8
prompt", "5b Replace deploy.ts:14 prompt", "5c Replace
configure.ts:22 prompt" instead of one mega-step.

## What to record

The walkthrough table. If every cell is ✓, write
"validate-runs-after-each-step: ✓" and move to the next
validator. If any cell is ✗, fix the plan **before** moving on.
