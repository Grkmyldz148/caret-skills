# surface-taxonomy

> Reference table. Read once when you start; refer back when
> grouping inventory entries.

The taxonomy is finite. Every CLI surface fits one of these
categories. Memorising the list lets you group an inventory in
minutes.

## The 12 surfaces

### prompt
Anything that asks the user a question and waits for keyboard
input. Includes:
- text input
- password input (hidden)
- yes/no confirm
- single-select from a list
- multi-select from a list
- numeric input
- form (multiple fields shown together)
- autocomplete (typed input + filtered list)
- search (typed input over a dataset)
- editor (multi-line text input)

### spinner / progress
Anything that shows the system is working. Includes:
- single-task spinners (ora-style)
- determinate progress bars (10%, 20%, …)
- indeterminate progress bars (oscillating)
- step indicators where each step is one task

### splash / banner
The opening surface of the CLI on first run or when called with
no args. Includes:
- ASCII art logos
- version + tagline blocks
- feature list / usage hint
- "first run" introductory text

### error
Surfaces that report a failure. Includes:
- inline error messages (one-line)
- error blocks with stack traces
- "command not found" / unknown-arg errors
- validation errors after a prompt
- exit-with-cause messages

### table / list
Multi-row data display where rows have structure. Includes:
- aligned tables (`name | size | mtime`)
- bullet lists with parallel structure
- two-column lists (key + description)
- nested trees (`├─ a ├─ b └─ c`)

### step indicator
Multi-step processes shown together. Different from spinner in
that the user sees all steps, not just the current one. Includes:
- boot-style sequential lists ("Loading config…done", "Starting
  server…done")
- listr-style task trees with parallel + sequential branches
- multi-stage deploy progress

### log
Generic line-by-line output, usually verbose / debug mode.
Includes:
- timestamped log lines (`[12:01:33] connected`)
- level-prefixed lines (`INFO connected`)
- raw `console.log` output the CLI sometimes does
- dry-run output that mirrors what would happen

### key-value
Display of `name: value` pairs without column alignment. Includes:
- summary blocks after a successful operation
- config dumps
- "current state" snapshots
- diff'd settings (`old: x | new: y`)

### diff
Before/after data display where the contrast is the point.
Includes:
- file diff output (--- +++ - +)
- config diff output (- prev: x / + new: y)
- changeset previews

### toast / notification
Transient, dismissible info that doesn't fit inline. Includes:
- "Copied to clipboard" floats
- "Saved" confirmations
- non-blocking warnings ("Cache was stale, rebuilt")
- background-task completion notices

### help / usage
Surfaces invoked by `--help`, unknown args, or
"command-not-found". Includes:
- top-level usage banner
- command-list with one-line descriptions
- per-command help (args, flags, examples)
- argument suggestions ("Did you mean…?")

### link
Anywhere the CLI tells the user to click / open a URL. Includes:
- "Open https://…/auth to log in"
- "Docs: https://…"
- "Report bugs at https://…"
- OAuth / device-code flows

## Mapping each to a Caret component

| Surface           | Default Caret component |
| ----------------- | ----------------------- |
| prompt            | `prompt.<variant>`      |
| spinner / progress | `spinner` (function) or `progress` |
| splash / banner   | `splash` or `banner`    |
| error             | `error`                 |
| table / list      | `table` / `list` / `tree` |
| step indicator    | `boot` or `step-list`   |
| log               | `log`                   |
| key-value         | `key-value`             |
| diff              | `diff`                  |
| toast / notification | `toast`              |
| help / usage      | (no default — see below) |
| link              | `link`                  |

## What about help / usage?

Help/usage is the one surface Caret doesn't ship a single
component for, because it's tightly coupled to the CLI parser
(commander, yargs, oclif). Recommend the user keep their
parser's built-in help renderer, but **theme it through Caret's
paint/typography helpers** so the colours and tracking match.

If the user is on commander, that's:

```ts
import { paint, tokens } from './caret'

program.configureHelp({
  formatHelp: (cmd, helper) => {
    // ... use paint / tokens to colour the output
  }
})
```

This keeps the parser's parsing logic intact and just changes
the visual layer.

## When a surface fits two categories

It probably is two surfaces. Examples:

- "Spinner that prints sub-progress lines" = spinner + log
- "Form with inline validation errors" = prompt.form + error
- "Boot list that ends with a key-value summary" = step indicator
  + key-value

Note both in the inventory; map both in the mapping step.
