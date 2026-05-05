# pipeline-inventory

> First step. Read this when the user asks for a new component and
> before you read any other rule.

You cannot design a Caret-compliant component without knowing what
it is *for*. Components designed from a one-liner ("make a `<Banner>`")
either miss the actual need or balloon into kitchen-sink APIs.
Two minutes here saves an hour of redesign later.

## What to extract from the user's request

Run a single short pass before opening any other rule. You are
filling in a four-line spec.

```
Component name : ___
Lifecycle      : { immediate-render | animated | interactive | async-resolution }
Inputs         : ___ (props the caller passes)
Resolves to    : ___ (what the caller does with the result, if any)
```

If you cannot answer all four lines from the user's request, **ask
one clarifying question before continuing**. Do not guess.

## Lifecycle taxonomy

These four labels cover everything Caret currently ships. Pick one.
The choice constrains the API and the test strategy downstream.

- **immediate-render** — pure function of props, draws once, never
  changes. Examples: `<KeyValue>`, `<Banner>`, `<Divider>`.
- **animated** — draws over time without user input. Examples:
  `<Spinner>`, `<Splash>`, `<Typewriter>`, `<Reveal>`.
- **interactive** — accepts keyboard input, may cancel/submit.
  Examples: `<Prompt.Text>`, `<Form>`, `<Search>`, `<Modal>`.
- **async-resolution** — kicks off async work, renders progress,
  resolves to a value. Examples: the `spinner('label', task)`
  pattern, `<DownloadProgress>`.

A component that needs *two* of these (e.g. interactive + async) is
almost always two components composed; flag it and split.

## Examples

User says: *"I need a thing that shows a list of changed files like
git status, with the file types coloured."*

```
Component name : <FileStatus>
Lifecycle      : immediate-render
Inputs         : files: { path: string; status: 'M'|'A'|'D'|'R'|'?' }[]
Resolves to    : nothing — it just renders
```

User says: *"A confirm prompt but with a third option."*

```
Component name : <Prompt.Triage>  (NB: prompt.* namespace)
Lifecycle      : interactive
Inputs         : label, options: [string, string, string], default?: 0|1|2
Resolves to    : index of chosen option (number)
```

User says: *"Show progress while we upload N files."*

```
Component name : <UploadProgress>
Lifecycle      : async-resolution
Inputs         : files: File[]; uploader: (f: File) => Promise<void>
Resolves to    : { ok: number; failed: number }
```

## What to do with the answer

Write the four-line spec at the top of your scratch buffer (or the
first line of the conversation reply if you are not using a buffer).
Every later step refers back to it. If the spec changes mid-build —
because the user clarified, or you found a registry component that
already covers it — update the four lines, do not patch downstream.

Move to `pipeline-registry-check.md`.
