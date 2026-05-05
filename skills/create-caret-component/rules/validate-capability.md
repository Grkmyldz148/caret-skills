# validate-capability

> Run after manifesto. Catches the "looks great on iTerm, broken
> in CI logs" failure mode.

A Caret component must render acceptably across the four capability
axes:

| Axis     | Tier 1     | Tier 2 | Tier 3 |
| -------- | ---------- | ------ | ------ |
| color    | truecolor  | 256    | 16 / mono |
| unicode  | full       | basic  | ASCII only |
| tty      | TTY        | piped  | (n/a) |
| motion   | enabled    | reduced | disabled |

Tier 1 is iTerm + macOS Terminal + modern Linux. Tier 3 is CI logs,
Docker exec output piped to a file, screen readers. The component
must work — not just "not crash" — at all three tiers on every axis.

## How to verify

The four required tests from `pipeline-test.md` cover most axes.
This validator is for the rest.

### Color axis

1. Run the component with `FORCE_COLOR=0` (forces 16-color):
   ```bash
   FORCE_COLOR=0 node -r tsx your-test-script.ts
   ```
   The output should have the same shapes (glyphs in the right
   places, layout intact) just with fewer / less precise colours.
2. Run with `NO_COLOR=1`:
   ```bash
   NO_COLOR=1 node -r tsx your-test-script.ts
   ```
   The output should have **no ANSI codes at all** but still
   convey the semantic distinctions (e.g. `[ERROR]` prefix
   instead of red text).

If either tier renders something that's unreadable or misleading —
e.g. an error message that looks the same as a success message
without the colour distinguishing them — the component fails this
check. Fix by adding a structural difference (prefix, glyph, label)
that survives colour stripping.

### Unicode axis

1. Set `LANG=C` and re-render:
   ```bash
   LANG=C node -r tsx your-test-script.ts
   ```
   Every Unicode glyph should fall back to its ASCII sibling. If
   the output contains literal `?` or boxes (mojibake), the
   component is reading a token that doesn't have an `_ascii`
   sibling — fix the token, not the component.

### TTY axis

1. Pipe the output through `cat`:
   ```bash
   node -r tsx your-test-script.ts | cat
   ```
   The output should be plain text that reads top-to-bottom. No
   cursor moves, no overprints, no spinners. If the output writes
   the spinner frames as separate lines (`⠋⠙⠹⠸⠼⠴`), the component
   is animating without checking `capability.tty` — fix it.

### Motion axis

1. Set `CARET_REDUCE_MOTION=1`:
   ```bash
   CARET_REDUCE_MOTION=1 node -r tsx your-test-script.ts
   ```
   Animated components should render a single static frame with
   the same final-state semantics. A spinner under reduced motion
   is "Loading…" then "✓ Loaded" — no intermediate frames.

## What to look for in the output

For each axis × tier combination:

- **Layout intact?** Boxes still align, columns still align,
  no overlapping characters.
- **Semantics preserved?** Errors still distinguishable from
  successes. Selected items still distinguishable from
  unselected.
- **No mojibake?** No `?` boxes, no replacement characters,
  no half-printed escape codes.

If any combination fails, fix at the **component** level (add the
missing branch) — never at the user level (telling users to
"upgrade your terminal" is the wrong fix).

## Common failures

- **Spinner glyphs in a non-TTY**: `⠋⠙⠹⠸⠼⠴` written one per line
  to a log. Fix: `if (!capability.tty) return <Text>{label}</Text>`.
- **Box-drawing in 16-color terminal**: Ink renders the box but
  the colours are wrong because the component uses `#5882f7`
  directly. Fix: use `paint.accent`.
- **Selected-item highlight invisible under NO_COLOR**: relies on
  bg-color to indicate selection. Fix: prefix with a glyph
  (`▍ Selected option`).
- **Animation frames printed sequentially in pipe mode**: missing
  `capability.tty` check before timer setup.

## What to record before moving on

A short bullet list of which tier × axis combinations you
verified. Example:

> Verified:
> - color truecolor / 256 / 16 / mono ✓
> - unicode full / ASCII ✓
> - tty / piped ✓
> - motion enabled / reduced ✓

If you skipped a tier ("can't test 256-color locally"), say so
explicitly. Don't claim coverage you didn't verify.

Move to the next validator.
