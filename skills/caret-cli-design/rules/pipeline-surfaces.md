# pipeline-surfaces

> Read after inventory. Before mapping.

You have a flat list of file:line entries. Now group them by
**surface type** so the next step can map each surface to one
Caret component (or a small set of related ones).

## The grouping

Every CLI surface in the inventory belongs to exactly one of these.
Open `surface-taxonomy.md` for the full reference; the grouping
below is the quick version.

- **prompt** — anything that asks the user a question
- **spinner / progress** — anything that shows work in flight
- **splash / banner** — opening art, version banner, hero block
- **error** — error messages, including stack traces
- **table / list** — multi-row data display
- **step indicator** — multi-step progress (boot, init,
  deploy phases)
- **log** — generic line-by-line output (verbose mode, debug)
- **key-value** — settings dump, status dump, summary
- **diff** — before/after data
- **toast / notification** — transient, dismissible info
- **help / usage** — `--help` output, command-not-found,
  argument-help
- **link** — anywhere the CLI tells the user to "open
  http://..."

If a surface in your inventory doesn't fit any of these,
either:

1. The taxonomy is incomplete (rare — flag for refinement),
2. The surface is doing two things and should split, or
3. The surface is novel enough to need a custom component
   (sister skill: `create-caret-component`).

## Worked example

Inventory from `pipeline-inventory.md`:

```
src/index.ts:14   — banner ASCII art on first run
src/index.ts:22   — chalk.bold.cyan splash with version
src/commands/init.ts:8   — inquirer prompt for project name
src/commands/init.ts:23  — ora spinner during scaffolding
src/commands/init.ts:48  — chalk.green ✓ + summary list
src/commands/deploy.ts:12 — chalk.yellow warn before deploying
src/commands/deploy.ts:31 — listr task tree for deploy steps
src/commands/deploy.ts:67 — chalk.red error block + stack trace
src/lib/log.ts:5  — wraps console.log with timestamp
```

Grouped:

| Surface          | Inventory entries |
| ---------------- | ----------------- |
| splash / banner  | `index.ts:14`, `index.ts:22` |
| prompt           | `init.ts:8` |
| spinner          | `init.ts:23` |
| key-value        | `init.ts:48` (the summary list) |
| step indicator   | `deploy.ts:31` (listr task tree) |
| toast            | `deploy.ts:12` (transient warn) |
| error            | `deploy.ts:67` |
| log              | `log.ts:5` (CLI-wide log helper) |

Notice:
- "splash" and "banner" are merged — they are the same surface
  (opening identity), the CLI just split it across two lines.
- "summary list with green ✓" is a **key-value** surface, not a
  table — there's no row data, it's a list of completed items.
- The listr task tree is a **step indicator**, not a spinner;
  spinners are single-task, step indicators are multi-task with
  ordering.

## When in doubt about grouping

Ask: "What is the user's mental model when they see this?"

- Are they being asked to choose? → prompt
- Are they being told to wait? → spinner / progress
- Are they being told the result of something? → error / toast /
  key-value (depending on what kind of result)
- Are they being walked through a process? → step indicator

If the answer is "all of the above" for a single surface, the
surface is conflating concerns. Note it; the mapping step will
likely split it.

## What to record before moving on

The grouped inventory, as a markdown table. This is one of the
deliverables the user sees — invest in clarity. Two columns:
surface type, inventory entries.

Move to `pipeline-mapping.md`.
