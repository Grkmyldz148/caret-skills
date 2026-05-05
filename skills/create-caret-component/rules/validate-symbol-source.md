# validate-symbol-source

> Run after `validate-manifesto`. Catches the silent symbol-drift
> failure mode.

Every glyph in a Caret component must come from `tokens.symbols.*`.
Inline glyphs are forbidden, even if they "look the same" as the
token. The reason: themes override tokens; a hardcoded glyph
ignores the theme and creates inconsistency that's invisible at
review time.

## What counts as a "glyph"

Anything outside ASCII 0x20-0x7E that the component renders. This
includes:

- Box-drawing: `─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼`
- Arrows: `← → ↑ ↓ ↔ ↕ ⇒ ↗ ↘`
- States: `✓ ✗ ⚠ ℹ ●`
- Bullets / dots: `• ▸ ◆ ◊ ▍ ▎`
- Cursors: `▍ ▎ █ ▌ ▐`
- Spinners: `⠋ ⠙ ⠹ ⠸ ⠼ ⠴`
- Half-blocks / shading: `░ ▒ ▓ █`
- Anchors: `›  ❯ ⌘`
- Anything CJK / emoji / math symbols / other Unicode

## Check

```bash
# Find non-ASCII characters in the component file
grep -nP '[^\x00-\x7F]' registry/components/<your-component>.tsx
```

For every line that matches, open the file and confirm:

1. The character is inside a string literal (not a comment — those
   don't render).
2. The string literal is the value of a `tokens.symbols.*` lookup,
   **not** a `<Text>{'<literal>'}</Text>` or template literal.

If a non-ASCII character appears outside `tokens.symbols`, it's a
violation.

## Common violations

```tsx
// ❌ Inline glyph — even though ✓ is the success token's value,
//    this hardcodes it, bypassing the theme.
<Text color={paint.success}>{'✓'} Done</Text>

// ✅ Use the token.
<Text color={paint.success}>
  {tokens.symbols.state.success} Done
</Text>
```

```tsx
// ❌ Inline arrow.
<Text>{`Step ${i+1} → ${name}`}</Text>

// ✅ Use the token.
<Text>{`Step ${i+1} ${tokens.symbols.arrow.right} ${name}`}</Text>
```

```tsx
// ❌ Inline box-drawing.
<Text>{`┌─ ${title} ─┐`}</Text>

// ✅ Use the layout primitive (Box border).
<Box borderStyle="single" paddingX={1}>
  <Text>{title}</Text>
</Box>
```

That last one is special — box-drawing is **never** done by string
concatenation in Caret. Always use Ink's `<Box borderStyle>` or the
`registry/components/divider.tsx` component, which read from the
token system.

## What if the token doesn't exist?

If you need a glyph the registry doesn't have, do **not** inline
it. Instead, edit `registry/tokens/symbols.ts` to add the new
glyph (with an ASCII fallback), then reference it from your
component. See `pipeline-tokens.md` "extending the symbol set"
for the procedure.

The reason this is enforced is that the symbol set is the
component-set's signature. A `<Banner>` component with an inline
✦ that ships before the symbol set has ✦ creates a precedent;
the next author writes `<Splash>` with their own ✦, slightly
different. Six months later the system has three different ✦s
and no theme can fix it.

## Edge case: text content from props

If the component renders `props.label` or any other user-supplied
string, it's not subject to this validator. The user is allowed
to put whatever character they want in their label. The validator
applies to glyphs the **component** chooses.

## What to record

Pass / fail. If fail, list each violation as `<file>:<line> —
<inline glyph> should be <token reference>`.
