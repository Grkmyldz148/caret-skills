# pipeline-mapping

> Read after surfaces. This is where the plan starts to take shape.

For each grouped surface, pick the Caret component that replaces
it. The reference is `component-catalog.md` (and `npx caret-cli@
alpha list` for the live truth).

## The decision per surface

For each surface in your grouping, output one row:

```
<surface>  →  <Caret component>  +  <token customisations, if any>
                                  +  <"new" if creating, else file path>
```

Three possible verdicts:

1. **Direct map** — registry has a component for this surface.
   Example: prompt → `prompt.text` / `prompt.confirm` / `prompt.select`.
2. **Composed map** — surface needs two registry components together.
   Example: error block → `error` (the component) wrapped in
   `panel` (boxed container).
3. **No fit** — registry doesn't ship this surface yet. Flag for
   the sister skill `create-caret-component`. Do not pretend
   there's a fit.

## Worked example (continuing from `pipeline-surfaces.md`)

| Surface          | Caret component | Notes |
| ---------------- | --------------- | ----- |
| splash / banner  | `splash`        | Customise `theme.brand` to acme palette |
| prompt           | `prompt.text`   | Direct |
| spinner          | `spinner` (function form) | Direct |
| key-value        | `key-value`     | Direct |
| step indicator   | `boot` or `step-list` | `boot` if sequential, `step-list` if parallel |
| toast            | `toast`         | Direct |
| error            | `error`         | Maybe wrap in `panel` for stack traces |
| log              | `log`           | Replace `console.log + timestamp` with `log.info` etc. |

Each row produces one or two `npx caret-cli@alpha add <name>`
commands later in `pipeline-adoption-order.md`.

## When the registry has variants

Some Caret components have multiple shapes (`prompt.text`,
`prompt.confirm`, `prompt.select`, …). Pick by inspecting the
existing surface:

- If it's `inquirer.prompt({ type: 'input', ... })` →
  `prompt.text`
- If it's `inquirer.prompt({ type: 'confirm', ... })` →
  `prompt.confirm`
- If it's `inquirer.prompt({ type: 'list', ... })` →
  `prompt.select`
- If it's `inquirer.prompt({ type: 'checkbox', ... })` →
  `prompt.multi-select`

Same logic for spinner-likes:
- `ora()` for one task → `spinner` function
- `listr` task tree → `boot` (sequential) or `step-list`
  (parallel-aware)
- Custom progress bar → `progress`

## Composed mappings (when one isn't enough)

Some surfaces want a wrapper:

| Surface                                   | Composition |
| ----------------------------------------- | ----------- |
| Error message + stack trace               | `panel` containing `error` |
| Banner with logo + version + tagline      | `splash` (which composes banner + key-value internally) |
| Confirm prompt before destructive action  | `prompt.confirm` with `theme.intent='danger'` |
| Long-running task with intermediate logs  | `spinner` function + `log.info` between phases |

Composed mappings are **two `caret-cli add` commands** in the
adoption sequence. List both.

## When there's no fit

Don't paper over it. If the user has a surface that the registry
doesn't cover — say a custom interactive map view, or a
spectrogram, or a TUI dashboard with multiple panels — write:

```
| dashboard view | NO FIT — registry doesn't cover multi-panel TUIs.
                  Recommend: keep current implementation, or invoke
                  `create-caret-component` skill for a Caret-native
                  rewrite. |
```

Listing the gap honestly is more useful than a forced fit.

## Token customisations to call out

For each direct/composed map, note any token customisation you'd
recommend:

- Brand colour different from Caret default → `theme.color.accent`
  customisation
- Symbol set preference (e.g. ASCII-only output for legacy
  terminals) → `theme.symbols.preset = 'ascii'`
- Spacing preference (compact vs roomy) → `theme.spacing.scale`

Don't recommend customisations the user didn't signal a need for.
"Caret defaults are good" is the right answer for most rows.

## What to record before moving on

A complete mapping table. Every surface from the previous step
appears in exactly one row. Each row shows:

- the Caret component name (or "NO FIT"),
- any token customisations,
- a brief reason if non-obvious.

Move to `pipeline-tokens.md`.
