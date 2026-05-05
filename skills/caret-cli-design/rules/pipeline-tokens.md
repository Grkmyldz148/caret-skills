# pipeline-tokens

> Read after mapping. The tokens are how the user's brand survives
> the migration.

Caret ships with tuned defaults for accent, semantic colours,
symbols, spacing, and motion. Most CLIs should take the defaults.
The exceptions are:

1. **Brand accent** — the one colour that's "theirs."
2. **Symbol preset** — ASCII-only sometimes wins.
3. **Reduced motion** — when the CLI runs primarily in CI.

Decide each. Default is "leave alone."

## Brand accent

Find the brand colour. Sources, in priority order:

1. **README hero / logo** — if there's an SVG logo, the primary
   stroke colour is usually the brand.
2. **`package.json` keywords or homepage** — sometimes references
   a marketing site whose CSS has the palette.
3. **Existing `chalk.<colour>` calls** — the colour the CLI uses
   most for accents (not errors, not warnings — pure accents).
   `chalk.cyan` and `chalk.blue` are the most common signals.
4. **Ask the user**.

Once you have the brand hex, the customisation is:

```ts
// In the user's caret.config.ts (or wherever they wire theme)
import { defaultTheme, mergeTheme } from './caret/theme'

export const theme = mergeTheme(defaultTheme, {
  color: {
    accent: { hex: '#ff5722', ansi256: 202, ansi16: 'red' }
  }
})
```

The three forms (truecolor / 256 / 16-color) come from running the
hex through Caret's palette tool. If the user is in a hurry, just
the hex is fine — Caret's palette helper computes the fallbacks.

## Symbol preset

Three presets ship:

- **`unicode` (default)** — `✓ ✗ ⚠ ▍ → •` — best on modern
  terminals.
- **`ascii`** — `[OK] [X] [!] | -> *` — universal compatibility,
  preferred for legacy / CI / SSH-to-old-server scenarios.
- **`compact`** — minimal whitespace, single-char symbols only —
  for narrow terminals (40 cols) or status bars.

Recommend `ascii` if the CLI explicitly markets to legacy
environments (sysadmin tools, on-prem ops, embedded). Otherwise
keep `unicode`.

## Reduced motion

Three settings:

- **`auto` (default)** — respects `CARET_REDUCE_MOTION`,
  `NO_MOTION`, OS preference.
- **`always`** — animations always render statically. Good for CI
  CLIs where animation is just noise in logs.
- **`never`** — animations always render, ignore environment.
  Almost always wrong. Don't recommend this.

If the CLI's primary use is `npm run ci` / GitHub Actions,
recommend `always`. Otherwise `auto`.

## Anti-patterns

- **Recommending five colour overrides.** The semantic palette
  (success/warning/error/info) is tuned together. Override
  `accent` only; let the rest stay coordinated.
- **Recommending a symbol override per-component.** That's not
  what tokens are for. If the user wants a different cursor,
  override `theme.symbols.cursor` once; don't pass a `cursor`
  prop to `<Prompt>`.
- **Recommending font / typography customisations.** Caret
  doesn't theme typography — terminals don't expose enough of
  the font. The `tracking()` and `caps()` helpers are static
  utilities, not theme tokens.

## What to record before moving on

A short token decisions block:

```
Token decisions:
- Brand accent: #ff5722 → theme.color.accent (matches logo)
- Symbol preset: unicode (default — terminals are modern)
- Motion: auto (default — works in CI and local)
```

Three lines, three decisions, with one-line rationale each.

Move to `pipeline-adoption-order.md`.
