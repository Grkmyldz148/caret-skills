# pipeline-adoption-order

> Read after tokens. This is the only step that turns "what to do"
> into "what to do *first*."

A plan that lists 12 changes in random order is one the user will
not execute. They'll do step 1, hit a wall because step 4 doesn't
compile without step 7, and abandon. Topological sort matters.

## The constraint

After every step in the adoption sequence, the CLI must:

- compile (no TypeScript errors),
- run (no runtime errors on the happy path),
- produce output the user recognises (not regress visibly to
  the user's existing customers).

If a step violates any of those, reorder until it doesn't. Use
`validate-runs-after-each-step.md` to confirm.

## Recommended order (default)

Apply this template, then adjust based on dependencies you find.

1. **Install caret-cli + scaffold theme** — copy
   `theme/`, `tokens/`, `lib/paint.ts`, `lib/capability.ts`. No
   user-facing change yet; just sets up the palette.
2. **Set the brand accent** — apply the token customisation.
   Still no user-facing change unless something already imports
   from `./caret/theme`.
3. **Replace logging** — `npx caret-cli@alpha add log`,
   then update `console.log → log.info`, `console.warn →
   log.warn`. Low-risk, high-coverage. Every CLI has logs.
4. **Replace error handling** — `npx caret-cli@alpha add error`,
   then update error printers. Often shares helpers with logging
   so this comes second.
5. **Replace prompts** — `npx caret-cli@alpha add prompt`, then
   migrate `inquirer.prompt` calls one at a time. Each migration
   is local; do them in any order.
6. **Replace progress** — `npx caret-cli@alpha add spinner`,
   then `add boot` or `add step-list` if needed. Replace `ora`
   and `listr` calls. Higher-effort because async resolution
   patterns may need refactoring.
7. **Replace splash / banner** — `npx caret-cli@alpha add
   splash`, replace the opening art. Usually a one-shot rewrite.
8. **Replace key-value summaries** — `npx caret-cli@alpha add
   key-value`. Mostly cosmetic; do last.
9. **Add toasts** — only if the CLI has transient
   notifications today. Most don't.

## Why this order

- **Theme first** because every later step depends on it.
- **Logging early** because it touches every file and is the
  least likely to break behaviour. Wins early confidence.
- **Errors next** because error handling shares helpers with
  logging, and getting it wrong hurts most.
- **Prompts mid-stack** because they're isolated — each prompt is
  one call site. Can do incrementally.
- **Progress later** because it's the highest-risk migration:
  async patterns, race conditions, SIGINT handling.
- **Splash last visible** because it's pure cosmetics. Visible
  but trivial; saves the dopamine hit for the end.

## Dependency cases that override the default

Reorder when:

- **The error printer reaches into the spinner state.** Move
  spinner before error.
- **Splash already uses chalk in a way that imports the rest of
  the chalk-based codebase**, fix splash early so chalk can be
  removed cleanly.
- **The CLI has a single big "render" function that does
  everything**, the migration is a rewrite, not incremental —
  flag it and write a different plan (rewrite + landing PR).

## Per-step structure

Each step in your plan has:

1. **Title** — what the step does, in one line.
2. **Commands** — the exact `npx caret-cli@alpha add ...`
   incantations.
3. **Files touched** — list of paths the user will edit.
4. **Verify** — what to check after to confirm the step works.

Example:

```
Step 3 — Replace logging

Commands:
  npx caret-cli@alpha add log

Files touched:
  src/lib/log.ts                 (replace contents with caret import)
  src/index.ts                   (replace `console.log` with `log.info`)
  src/commands/init.ts           (same)
  src/commands/deploy.ts         (same)

Verify:
  - `pnpm tsc --noEmit` passes
  - `acme-cli init` produces output that still says what it said before
  - `acme-cli init --verbose` still works
```

## What to record before moving on

The full step-by-step adoption plan. Each step in the structure
above. Together this is the bulk of the deliverable.

Move to `pipeline-risks.md`.
