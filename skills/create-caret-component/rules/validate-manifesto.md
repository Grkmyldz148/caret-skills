# validate-manifesto

> Run this last. It is the floor. If any check fails, the component
> is not ready, regardless of how nice the rest looks.

The Caret manifesto is the contract that lets dozens of components
look like one design system. A component that violates any of these
checks is a "Caret-shaped Ink demo" — the failure mode this skill
exists to prevent.

## Checks

For each, grep / read the component file and answer yes or no.

### 1. Semantic colours only

```bash
# Forbidden patterns:
grep -nE "chalk\.|#[0-9a-fA-F]{3,8}\b|ansi\([0-9]" <file>
```

Any match = fail. Colours come from `paint.<semantic>` or
`tokens.colors.<semantic>`.

### 2. NO_COLOR honoured

The component does not need to read `process.env.NO_COLOR`
directly — the `paint()` helper handles it. But:

- If the component uses `paint.*` or `theme.color.*`, ✓ implicitly.
- If the component imports `chalk` or uses raw escape codes, ✗.

```bash
grep -nE "import .*chalk|from ['\"]chalk['\"]" <file>
```

Any match = fail.

### 3. Capability fallbacks

For every Unicode glyph (anything outside ASCII 0x20-0x7E), the
component must branch on `capability.unicode` and fall back to an
ASCII alternative.

Quick check:

```bash
# Find non-ASCII characters in the file:
grep -nP '[^\x00-\x7F]' <file>
```

For each non-ASCII line, verify there's a matching `capability.
unicode ?` ternary or an upstream branch that selects the glyph.

### 4. No terminal bell

```bash
grep -nE "\\\\x07|\\\\u0007|String\.fromCharCode\(7\)" <file>
```

Any match = fail. The bell is banned.

### 5. No console writes

The component does not write to stdout or stderr directly.

```bash
grep -nE "console\.(log|warn|error|info|debug)|process\.std(out|err)\.write" <file>
```

Any match = fail. Output goes through Ink's renderer.

### 6. Symbols from token system

Every glyph in the render comes from `tokens.symbols.*`. Inline
literals are forbidden.

This one is hard to grep for — you have to read the JSX. Look for
`<Text>...</Text>` blocks containing characters outside `[a-zA-Z0-9
?!.,;:'"-+/=*\\(\\)\\[\\]\\{\\}<>]` and verify they came from a
token, not a literal.

Common offenders:
- `<Text>{'→'}</Text>` — should be `tokens.symbols.arrow.right`
- `<Text>{'•'}</Text>` — should be `tokens.symbols.bullet`
- `<Text>{'✓'}</Text>` — should be `tokens.symbols.state.success`
- `<Text>{'✗'}</Text>` — should be `tokens.symbols.state.error`

### 7. Motion respect

For animated components only, verify the timer (`setInterval` /
`setTimeout`) is gated by `capability.motion`:

```ts
useEffect(() => {
  if (!capability.motion) return  // <-- this line is required
  // ... timer setup
}, [...])
```

Missing the gate = fail.

### 8. Brand accent fixed

The component does not override `tokens.colors.accent`. Caret's
accent is brand identity; it does not adapt to user themes.

```bash
grep -nE "accent\s*:\s*['\"]" <file>
```

Any match outside the registry's central theme file = fail.

## What "fail" means

Mark the component as **not ready**. List every failed check in
the conversation reply with the file:line where the violation lives,
plus the fix. Example:

> ❌ `validate-manifesto` failed:
> - **Check 1 (semantic colours)**: `prompt.tsx:42` uses
>   `chalk.red(error)`. Replace with
>   `<Text color={paint.error}>{error}</Text>`.
> - **Check 6 (token symbols)**: `prompt.tsx:67` has literal
>   `'›'`. Replace with `tokens.symbols.prefix.focused`.

Do not move on until the user has applied the fixes (or accepted
the violations as deliberate, with reason — but that should be
extremely rare).

## When manifesto checks conflict

They don't, but if you think two checks contradict each other,
the resolution is:

1. Reread the component spec.
2. Reread `pipeline-render.md`.
3. If still stuck, the component is doing too much — split it.

If after splitting the conflict remains, file an issue against
`Grkmyldz148/caret`. Manifesto changes are above this skill's
authority.
