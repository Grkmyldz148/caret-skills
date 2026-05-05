# pipeline-tokens

> Read after registry check, before you decide on props.

Tokens are the **contract** between your component and the rest of
Caret. A component that picks its own colours, symbols, and spacing
will look correct in isolation and broken next to any other Caret
component. This step is where you commit to using the shared
vocabulary.

## The four token families

Every Caret component pulls from these and only these:

1. **`tokens.colors`** — semantic names (`accent`, `success`,
   `warning`, `error`, `info`, `muted`, `subtle`, `fg`, `canvas`).
   Never hex, never `chalk.<colour>`, never numeric ANSI codes.
2. **`tokens.symbols`** — Unicode glyphs grouped by purpose
   (`cursor`, `state.success`, `state.warning`, `prefix.focused`,
   `anchor`, `divider.h`, `divider.v`, `arrow.right`, …).
3. **`tokens.spacing`** — `0..6` scale that maps to character cells
   (terminal grid spacing). Do not invent half-cells.
4. **`tokens.typography`** — `tracking()` for letter-spacing
   (terminal "letter-space" emulated with whitespace), `caps()`
   for tracked CAPS, weight modifiers where the renderer supports
   them.

## How to pick

For each visible element your render produces, ask:

- **What semantic role does it carry?** — that's your colour token.
  Action confirmation → `success`. User-visible accent (brand,
  selection) → `accent`. Annotation → `subtle`. Body → `fg`.
  Errors → `error`. Defer → `muted`.
- **What glyph does it use?** — pick from `tokens.symbols`. If
  nothing fits, your component is asking for a new glyph; that
  goes through a registry edit (see "extending the symbol set"
  below), not into your component file.
- **How much space between elements?** — pick from `tokens.spacing`.
  In terminals this is character cells. `2` is two cells. `0` is
  no space.

## Worked example — `<FileStatus>` from inventory

Render: `<status-glyph> <path>` per file.

Token decisions:

| Element     | Token                              | Why |
| ----------- | ---------------------------------- | --- |
| 'M' glyph   | `tokens.symbols.state.modified`*   | Modified = warning-class state |
| 'M' colour  | `tokens.colors.warning`            | Yellow = "needs attention"     |
| 'A' glyph   | `tokens.symbols.state.added`*      | Added = success-class state    |
| 'A' colour  | `tokens.colors.success`            | Green = "good change"          |
| 'D' glyph   | `tokens.symbols.state.removed`*    | Removed = error-class state    |
| 'D' colour  | `tokens.colors.error`              | Red = "destructive"            |
| path text   | `tokens.colors.fg`                 | Body content                   |
| spacing     | `tokens.spacing[2]` between glyph and path | Standard list inset    |

\* Glyphs marked with an asterisk **are not yet in the registry**.
This is the case where your component asks for a new symbol. See
"extending the symbol set."

## Extending the symbol set

If you need a glyph the registry doesn't have, do **not** inline a
hardcoded character. Instead:

1. Open `registry/tokens/symbols.ts` (in the user's checkout, or
   read the published tarball under `registry/tokens/symbols.ts`).
2. Add the new symbol under the appropriate group, with:
   - a default Unicode character,
   - a 16-colour fallback (ASCII only, e.g. `'M'` → `M`),
   - a description comment explaining the role.
3. Reference it from your component as `tokens.symbols.<group>.<name>`.

This keeps the symbol vocabulary auditable and lets every Caret
component theme override the glyph in one place.

## Anti-patterns

These are caught by validators, but it's faster to not write them:

- `chalk.red(text)` — use `paint.error(text)`
- `process.stdout.write('✓')` — use `tokens.symbols.state.success`
- `'  '` (literal two-space indent) — use `' '.repeat(tokens.spacing[2])`
- `'#5882f7'` anywhere — there is no scenario where a hex literal is
  correct in a component
- `'\n\n'` (raw double newline) — use the appropriate spacing in
  the layout primitive (Box gap, etc.)

## What to record before moving on

In your scratch buffer, list every token you'll touch. One line
per token. The list will be 5-15 entries for a typical component.
This list is the *contract*: the validator at the end checks that
every glyph and colour in your final render comes from this list.

Move to `pipeline-api.md`.
