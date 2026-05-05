# pipeline-render

> Read after the API is fixed. This is where the JSX gets written.

The render step is mostly mechanical — translate tokens + props
into Ink JSX. The two places it goes wrong are:

1. **Capability branching** — assuming truecolor / unicode / TTY.
2. **Layout primitives** — using raw `\n` and string padding
   instead of `<Box>` / `<Text>`.

This rule covers both.

## Use Ink layout primitives

Caret renders through Ink. The two primitives you'll use 95% of the
time:

- **`<Box>`** — flexbox container. Has `flexDirection`, `gap`,
  `paddingX`, `paddingY`, `borderStyle`. Use it for any element
  that has children laid out spatially.
- **`<Text>`** — terminal text node. Has `color`, `bold`, `dim`,
  `italic` props. Wraps content into one or more lines.

Anti-patterns to avoid:

- `'  '` for indentation — use `<Box paddingX={2}>`.
- `'\n'.repeat(2)` between sections — use `<Box marginY={1}>`.
- `' '.repeat(width - text.length)` for right-alignment — use
  `<Box justifyContent="space-between">`.

If you find yourself string-concatenating layout, stop and
re-render with primitives. The terminal renderer handles widths,
wrapping, and cursor positioning correctly only when you do.

## Capability branching

Use `capability` from `registry/lib/capability.ts`. It exposes:

```ts
capability.color    // 'truecolor' | '256' | '16' | 'mono'
capability.unicode  // boolean
capability.tty      // boolean — false when piped
capability.motion   // boolean — false when CARET_REDUCE_MOTION or NO_MOTION
```

Branch on these explicitly. Do **not** assume; the user's terminal
might be CI logs, a screen reader, a Docker exec, or NixOS init.

### Colour branching

`paint()` from `registry/lib/paint.ts` already handles the
truecolor → 256 → 16 → mono fallback for every semantic colour.
Use it directly:

```tsx
import { paint } from 'caret-cli/registry/lib/paint'
// or in user code:
import { paint } from './caret/lib/paint'

// Renders the right shade of red regardless of capability.
<Text color={paint.error}>...</Text>
```

Do **not** branch on `capability.color` manually unless you need a
non-semantic effect (e.g. a custom gradient). The paint helper is
the contract.

### Unicode fallback

For every glyph, provide an ASCII fallback. The token file already
contains both:

```ts
tokens.symbols.state.success      // '✓'
tokens.symbols.state.success_ascii // 'OK'
```

Branch:

```tsx
const glyph = capability.unicode
  ? tokens.symbols.state.success
  : tokens.symbols.state.success_ascii
```

If a token doesn't have a `_ascii` sibling yet, you're adding one.
See `pipeline-tokens.md` "extending the symbol set."

### TTY (no-pipe) fallback

When `!capability.tty`, render a minimal text version. No boxes,
no spinners, no live regions.

```tsx
if (!capability.tty) {
  return <Text>{label}</Text>
}
```

For `async-resolution` components in pipe mode, log start, log
result, do not animate.

### Motion fallback

When `!capability.motion`, render a single static frame instead of
animating. This is one line in `useEffect`:

```tsx
useEffect(() => {
  if (!capability.motion) return  // never start the timer
  const id = setInterval(advance, cadence)
  return () => clearInterval(id)
}, [cadence])
```

## Render structure (template)

```tsx
import { Box, Text } from 'ink'
import { capability, paint, tokens } from 'caret-cli/registry'

export function MyComponent({ label }: MyComponentProps) {
  // capability branches up top
  if (!capability.tty) return <Text>{label}</Text>

  // glyphs resolved once
  const successGlyph = capability.unicode
    ? tokens.symbols.state.success
    : tokens.symbols.state.success_ascii

  // render
  return (
    <Box paddingX={tokens.spacing[2]}>
      <Text color={paint.success}>{successGlyph} </Text>
      <Text color={paint.fg}>{label}</Text>
    </Box>
  )
}
```

The shape — capability checks first, glyph resolution second,
render third — is the standard. Reviewers will scan for it.

## Forbidden in the render

- `process.stdout.write('\x07')` — terminal bell. Banned.
- `process.stdout.write` of any kind inside the render — Ink owns
  output. Side-channel writes corrupt the framebuffer.
- `console.log` — same reason.
- `setTimeout` for animation — use `useEffect` + `setInterval`
  scoped to mount.
- Raw escape codes (`'\x1b[2J'`, etc.) — Ink handles them; raw
  codes break the diff renderer.

## What to record before moving on

The render function, complete and tested mentally against:
- truecolor / 256 / 16 / mono terminal,
- Unicode / ASCII terminal,
- TTY / piped output,
- motion enabled / disabled.

If any of those four axes produces a broken render, fix before
moving on. The validators will catch it later but it's much
faster to catch it now.

Move to `pipeline-test.md`.
