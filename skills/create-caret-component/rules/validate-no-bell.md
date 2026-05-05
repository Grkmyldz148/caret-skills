# validate-no-bell

> Quick, single-purpose validator. Worth its own file because one
> stray bell makes the manifesto read as a lie.

The terminal bell (`\x07`, also written `BEL`, ``,
`String.fromCharCode(7)`) is **banned**. Caret components are
silent. Anything that wants to draw attention does it visually
(colour, glyph, position) — never aurally.

## Why

1. The bell is unreliable — half the terminals ignore it, the
   other half play the OS bell or flash the screen. There is no
   consistent UX.
2. It's user-hostile. People work in shared offices, on calls,
   in headphones. Beeping at them costs more than it gains.
3. Modern Caret CLIs ship with the ACS sound layer for
   considered, designed audio. The bell is the antithesis of
   that — it's the bell, regardless of context.

## Check

Single grep, single decision:

```bash
grep -nE "\\\\x07|\\\\u0007|String\\.fromCharCode\\(7\\)|BEL" \
  registry/components/<your-component>.tsx
```

Any match = fail.

The grep covers the four ways the bell character can appear in
JavaScript/TypeScript source. If you find one, it's a real
violation, even if "it's only in the error path."

## Why a separate file from `validate-manifesto`

`validate-manifesto.md` covers eight checks at once. It's easy for
one of them to be skimmed. The bell check is the one that, if
violated, makes the entire manifesto feel performative — so it
gets a dedicated 30-second pass.

## Fix patterns

If you wrote the bell to "draw attention to an error," replace
with:

```tsx
// Before — banned
process.stdout.write('\x07')
console.error('Failed')

// After — manifesto-compliant
<Text color={paint.error}>
  {tokens.symbols.state.error} Failed
</Text>
```

If you wrote the bell as a "completion notification" for a
long-running task, the right answer depends on context:

- For shell scripts, stop notifying. The user is watching
  the tail of output anyway.
- For interactive CLIs that the user might background, use a
  `notify` system call (macOS `osascript -e display notification`,
  Linux `notify-send`). Caret ships `notify` in
  `registry/lib/notify.ts` — use that helper, not the bell.

## What to record

If the check passes, write one line: "validate-no-bell: ✓".
If it fails, the component is not ready; fix and re-run.
