# pipeline-test

> Last pipeline step before the validators. Skipping it = the
> validators run against unverified code.

Two minimum tests per component. They are not for "coverage" —
they catch the manifesto violations a human reviewer would miss.

## Required tests

1. **NO_COLOR path** — rendering with `process.env.NO_COLOR='1'`
   produces output that contains the label and the ASCII glyph
   fallback, with **no ANSI escape codes**.
2. **No-TTY path** — rendering with stdout piped (not a TTY)
   produces minimal text output, no box-drawing, no live regions.

Optional but recommended for interactive / animated components:

3. **Keyboard interaction** — for interactive components,
   pressing Enter / Esc / arrow keys produces the right callback
   call.
4. **Reduced motion** — for animated components, setting
   `CARET_REDUCE_MOTION=1` produces a single-frame render.

## Tooling

Caret tests use `ink-testing-library`:

```ts
import { render } from 'ink-testing-library'
import { describe, it, expect, vi } from 'vitest'
import { MyComponent } from '../my-component'
```

Test files live as `*.test.tsx` next to the component source.

## NO_COLOR test (template)

```ts
describe('MyComponent', () => {
  it('honours NO_COLOR', () => {
    const prev = process.env.NO_COLOR
    process.env.NO_COLOR = '1'
    try {
      const { lastFrame } = render(<MyComponent label="Hello" />)
      const out = lastFrame()
      expect(out).toContain('Hello')
      // No ANSI escape codes
      expect(out).not.toMatch(/\x1b\[[0-9;]+m/)
    } finally {
      process.env.NO_COLOR = prev
    }
  })
})
```

## No-TTY test (template)

```ts
it('renders minimal text when stdout is piped', () => {
  // ink-testing-library renders to a fake stdout that is non-TTY by
  // default, so this test asserts that real-stdout-pipe behaviour
  // is the same shape.
  const { lastFrame } = render(<MyComponent label="Hello" />)
  const out = lastFrame() ?? ''
  // No box-drawing characters
  expect(out).not.toMatch(/[─│┌┐└┘├┤┬┴┼]/)
  // Label still present
  expect(out).toContain('Hello')
})
```

## Keyboard interaction test (template — interactive only)

```ts
it('calls onSubmit when Enter is pressed', async () => {
  const onSubmit = vi.fn()
  const { stdin } = render(
    <Prompt.Text label="Name" onSubmit={onSubmit} />
  )
  await new Promise((r) => setTimeout(r, 80)) // ink stdin attach
  stdin.write('Alice\r')
  await new Promise((r) => setTimeout(r, 30))
  expect(onSubmit).toHaveBeenCalledWith('Alice')
})
```

The 80 ms delay before the first write is required — ink subscribes
to stdin asynchronously after first render. Skipping it makes the
write race with the listener installation. Use `\r` for Enter,
`` for Esc, `[A`/`B`/`C`/`D` for arrows.

## Reduced motion test (template — animated only)

```ts
it('renders a single static frame under CARET_REDUCE_MOTION', () => {
  const prev = process.env.CARET_REDUCE_MOTION
  process.env.CARET_REDUCE_MOTION = '1'
  try {
    const { lastFrame } = render(<Spinner label="Loading" />)
    const first = lastFrame()
    // Wait past one cadence — output must not change
    return new Promise((r) => setTimeout(r, 200)).then(() => {
      expect(lastFrame()).toBe(first)
    })
  } finally {
    process.env.CARET_REDUCE_MOTION = prev
  }
})
```

## Anti-patterns in tests

- **Snapshot tests for ANSI output.** They turn into "approve any
  change" after the first regression. Assert against semantic
  shape (contains label, no escape codes) instead.
- **Setting `process.env` without restoring.** Leaks across tests
  in the same file. Always wrap in try/finally.
- **Testing the implementation.** If you find yourself asserting
  on `useState` calls or render counts, you're testing Ink, not
  your component.

## What to record before moving on

The two required tests, written and passing locally
(`pnpm vitest run <file>`). If either fails, the component fails
the build; do not move to validators with red tests.

Move to the validators (`validate-*.md`).
