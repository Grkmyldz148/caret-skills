# component-catalog

> Reference table. Snapshot of `npx caret-cli list` at the
> time of writing. The live command is the truth — verify before
> mapping.

This table is what you cite in the mapping step. If a row appears
here but `caret-cli list` shows it doesn't exist, the live list
wins (the registry has changed; this file is stale). If the live
list shows a row not here, add it.

## Last verified

Caret CLI version: `0.1.0`
Component count: 51

Run `npx caret-cli list` before publishing a plan, to make
sure recommendations are current.

## interactive (17)

| Component        | One-line role |
| ---------------- | ------------- |
| `prompt`         | text / password / confirm / select / multi-select / number |
| `spinner`        | loading with success/failure resolution |
| `splash`         | animated opening with logo, title, subtitle |
| `typewriter`     | character-by-character text reveal |
| `reveal`         | line-by-line text reveal |
| `boot`           | systemd-style sequential loader |
| `form`           | multi-field input layout with tab navigation |
| `modal`          | bordered overlay with action buttons |
| `toast`          | auto-dismissing inline notification |
| `autocomplete`   | fuzzy-search prompt over a list of options |
| `editor`         | multi-line text editor prompt with line numbers |
| `pager`          | scrollable viewport for long content with keyboard nav |
| `search`         | interactive fuzzy search/filter over a dataset |
| `tabs`           | tab bar navigation with ←/→ selection |
| `accordion`      | collapsible sections with ↑/↓ navigation |
| `toggle`         | boolean on/off switch with space to toggle |
| `scrollable`     | scrollable viewport container with scroll indicator |

## display (~25)

| Component        | One-line role |
| ---------------- | ------------- |
| `banner`         | one-line attention banner |
| `divider`        | horizontal hairline rule with optional label |
| `key-value`      | `key: value` pair list with optional alignment |
| `table`          | multi-column tabular data |
| `progress`       | determinate / indeterminate progress bar |
| `step-list`      | parallel-aware multi-step indicator |
| `log`            | level-prefixed line-by-line output |
| `error`          | error message with optional stack panel |
| `panel`          | bordered container for grouped content |
| `list`           | bulleted / numbered list |
| `badge`          | inline coloured tag |
| `tree`           | nested branching list (`├─ │ └─`) |
| `code-block`     | monospace code with optional language label |
| `diff`           | before/after data display |
| `kbd`            | keyboard shortcut chip |
| `tag`            | category / status tag |
| `breadcrumb`     | path-style navigation indicator |
| `link`           | URL with optional copy-to-clipboard |
| `file-status`    | git-style file status list |

## utility (helpers, not components)

| Module           | Role |
| ---------------- | ---- |
| `paint`          | semantic colour helpers (paint.success, paint.error, …) |
| `capability`     | runtime detection (color depth, unicode, TTY, motion) |
| `motion`         | easing + frame-loop helpers |
| `notify`         | OS notifications (osascript, notify-send, etc.) |
| `text-to-art`    | figlet-style ASCII headlines |
| `image-to-art`   | image → ASCII art renderer |
| `typography`     | tracking() and caps() helpers |

## How to use this catalog in mapping

When you grouped your inventory in `pipeline-surfaces.md`, you
labelled each entry with a surface type from
`surface-taxonomy.md`. The default Caret component for each
surface type is in this catalog under the matching column.

When in doubt:

1. Look at the inventory entry's actual behaviour.
2. Find the closest match here by *role*, not by name.
3. If the closest match is < 70% close, mark "NO FIT" in the
   mapping and recommend the `create-caret-component` skill.

## What's missing (snapshot — verify against live)

The registry doesn't currently ship:

- `dashboard` (multi-panel TUI)
- `chart` (line / bar / sparkline)
- `wizard` (multi-step guided form across multiple screens)
- `live-tail` (auto-scrolling log viewer with filters)
- `command-palette` (⌘K-style fuzzy command finder)

If the user's CLI has any of these surfaces, the mapping is
"NO FIT — consider authoring via `create-caret-component`."
