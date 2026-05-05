# pipeline-risks

> Read after adoption-order. Surfaces what the user gives up.

A Caret adoption plan is honest about what changes — including the
parts the user might not want to lose. Glossing over these makes
the plan look easy and then surprises them mid-migration.

## The standard risk list

Walk the inventory and check for each. Mark whichever apply.

### Manifesto incompatibilities

These are non-negotiable — Caret won't ship them. The user has to
accept losing them to adopt Caret.

- **Terminal bell** — if the existing CLI uses `\x07` /
  `String.fromCharCode(7)` anywhere, Caret will not preserve it.
  Identify which surfaces use it (often error handlers) and tell
  the user.
- **Custom hex colours** — Caret has semantic colours; the
  brand `accent` overrides one. Other custom hex colours
  (e.g. a special "info-blue" different from Caret's `info`)
  go away.
- **Inline emoji as state markers** — if the CLI uses
  emoji-state markers (🚀, 💥, ✨), they'll be replaced with
  semantic glyphs (✓, ✗, ⚠). Some users see this as a feature
  loss; tell them.
- **`console.log` of pre-formatted ANSI strings** — Caret
  components own their renderer. Pre-formatted strings (e.g.
  `chalk.bold(chalk.red('FAIL'))`) get replaced with structural
  components (`<Text bold color={paint.error}>FAIL</Text>`).

### Behaviour changes

These are not blockers, but they look different. Call them out.

- **Spinner cadence** — `ora` defaults to ~80ms; Caret's
  `spinner` defaults to ~120ms. Visibly slower if the user is
  watching. Configurable but defaults differ.
- **Reduced motion** — Caret respects `CARET_REDUCE_MOTION` /
  OS prefer-reduced-motion. CLIs that don't currently respect
  it will start doing so, which means CI logs become quieter.
  Almost always a good thing; warn the user anyway.
- **Pipe handling** — Caret components fall back to plain text
  when stdout isn't a TTY. CLIs that previously printed
  spinner frames into log files will now print plain text into
  log files. Better for ops, different from before.
- **NO_COLOR honouring** — Caret strictly honours NO_COLOR.
  CLIs that ignored it before will now produce uncoloured
  output for users who set it. Almost always a good thing.

### Implementation churn

Honest about effort:

- **Lines of code touched** — count from your inventory. If
  this is a 50+ file migration, say so.
- **Test fallout** — snapshot tests of CLI output **will**
  break. Migrating them is part of the work; budget for it.
- **Dependency removals** — `chalk`, `ora`, `inquirer`,
  `listr`, `figures` can typically all come out. Note them as
  "after the migration completes." Remove them only after
  all imports are gone.

## How to write the risk section

Use a markdown table:

```markdown
| Risk | What changes | Severity |
| ---- | ------------ | -------- |
| Terminal bell | `deploy.ts:67` uses `\x07` on error. Caret bans it. | manifesto-blocker |
| Custom hex `#5588ff` | `init.ts:48` uses chalk.hex; Caret's `info` is `#3b82f6`. Visible but small. | cosmetic |
| Spinner cadence | ~40ms slower. Visible to humans, invisible to logs. | low |
| Snapshot tests | 11 snapshot files reference current output; will need updates. | medium |
| Dependency removal | `chalk`, `ora`, `inquirer` can come out at the end. | low |
```

Severities:

- **manifesto-blocker** — user must accept this loss. Document.
- **cosmetic** — visible difference; user should know.
- **low / medium / high** — implementation effort.

## What to do if a manifesto-blocker is unacceptable

If the user can't lose (say) the bell, Caret might not be the
right system for them. Document this honestly:

> "If preserving the terminal bell is non-negotiable, Caret won't
> work for you — the manifesto rules it out. Consider keeping
> your existing CLI and using only Caret's component library
> selectively (e.g. just the `prompt` component) without the
> manifesto enforcement."

Selective adoption is fine. The CLI doesn't have to go all-in.

## What to record before moving on

The risk table. Three to seven rows is typical. If you have
zero rows, you missed something — every migration has at least
one risk.

Move to `pipeline-write-plan.md`.
