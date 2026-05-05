# pipeline-api

> Read after tokens. Before you write the render.

The component's props are its API. Once published, the API is hard
to change without breaking every caller. Spend time here so the
render step is a straight-line translation.

## Caret's prop conventions

Match these. Reviewers scan for them; deviating costs reader trust.

- **`label`** — the human-visible string the user sees. Required
  for all interactive components.
- **`value` / `defaultValue`** — controlled / uncontrolled pair.
  Pick one form per component. Most Caret prompts are
  uncontrolled (defaultValue) because shells don't have state.
- **`onChange` / `onSubmit` / `onCancel`** — event callbacks for
  interactive components. Names are exactly these — not
  `onConfirm`, not `onResolve`.
- **`disabled`** — boolean. When `true`, the component renders
  visibly but does not respond to input. (Most CLI components
  don't need this; only include if a flow disables it.)
- **`children`** — only for layout components (`<Box>`,
  `<Modal>`). Leaf components don't take children.
- **No `style` / `className` props.** Caret components are not
  themed externally; theming flows through `theme.color.*` and
  `tokens.symbols.*`. If a caller wants different colours, they
  override the theme, not the component.

## Lifecycle-specific shapes

Match the lifecycle from `pipeline-inventory.md`.

### immediate-render

```ts
type FileStatusProps = {
  files: { path: string; status: 'M'|'A'|'D'|'R'|'?' }[]
  /** Default: render all files. Cap at this many. */
  limit?: number
}

export function FileStatus(props: FileStatusProps): JSX.Element
```

No callbacks, no async. Pure function of props.

### animated

```ts
type SpinnerProps = {
  label: string
  /** ms between frames. Defaults to tokens.motion.spinner. */
  cadence?: number
  /** When prefers-reduced-motion is true, render a static frame. */
  reducedMotion?: 'auto'|'always'|'never'
}
```

Cadence and motion preferences are explicit props *with sensible
defaults*. Callers should rarely set them.

### interactive

```ts
type PromptTextProps = {
  label: string
  defaultValue?: string
  placeholder?: string
  validate?: (input: string) => string | null  // null = OK, string = error msg
  onSubmit: (value: string) => void
  onCancel?: () => void
}
```

Validation returns an error string or null — Caret doesn't use
exceptions for user input. `onCancel` is optional but recommended;
if absent, Esc is a no-op.

### async-resolution

The function-style API is preferred over a component for these:

```ts
async function spinner<T>(
  label: string,
  task: () => Promise<T>,
  opts?: { onSuccess?: string | ((value: T) => string); onError?: string }
): Promise<T>
```

Why a function? Async-resolution components naturally compose with
`await` in user code. A `<Spinner>` component would force users to
mount/unmount around the task, which is awkward in linear scripts.

## Defaults you must set

Components must work with **only the required props**. Required
means "the component cannot do its job without this." Everything
else has a default.

For a typical interactive prompt:
- Required: `label`, `onSubmit`
- Defaulted: `defaultValue: ''`, `placeholder: undefined`,
  `validate: () => null`, `onCancel: () => {}`

If you find yourself with 5+ required props, your component is
trying to do two jobs. Split it.

## TypeScript signature checklist

- [ ] All props are typed; no `any`, no `unknown` except where the
      caller really does pass arbitrary data.
- [ ] Callback prop types use `(...) => void` for fire-and-forget
      and `(...) => Promise<void>` for awaited callbacks.
- [ ] Async-resolution functions return `Promise<T>` where T is
      the resolved value, never `Promise<unknown>`.
- [ ] Generics are used for components that pass user data through
      (e.g. `prompt.select<T>` returns `T`, not `string`).

## What to record before moving on

A complete `Props` type definition, with JSDoc comments on each
prop explaining intent (not type — the type is in the signature).
This block goes at the top of your component file unchanged.

Move to `pipeline-render.md`.
