# pipeline-registry-check

> Run this immediately after `pipeline-inventory.md`. Before you
> write any code.

Re-implementing a component that already ships in the Caret registry
is the most common failure mode of this skill. The registry has
50+ components; it is faster to check than to remember.

## How to check

```bash
npx caret-cli@alpha list
```

This prints every component grouped by kind (`interactive`,
`display`, `utility`, `other`) with a one-line description. Read it.
Do not skim.

If you can't run the command (no network in this session), fall back
to the registry catalogue at the bottom of this file — but keep in
mind that the live `caret-cli list` is the source of truth and this
file may be one release behind.

## Decision tree

Compare your four-line inventory spec against every entry returned
by `caret-cli list`.

- **Exact match** — the registry already ships your component.
  Stop authoring. Tell the user:
  > "That's already a Caret component — `npx caret-cli@alpha add
  > <name>` will copy it into your project."
  Do not re-author.
- **Variant match** — the registry has the lifecycle and shape, but
  the user wants a different style (e.g. `<Spinner>` exists, user
  wants a "pulse" spinner). Two options:
  1. Recommend customising via tokens (preferred) — most variants
     come from a different `theme.symbol.spinner` set, not a new
     component.
  2. If tokens don't cover the variant, base your new component on
     the existing one. Cite the file path
     (`registry/components/spinner.tsx`) in your design notes.
- **Adjacent match** — the registry has something at the same
  *altitude* (`<Form>` for an interactive multi-field thing) but
  not your shape. Read the adjacent component's source for
  conventions (prop names, hooks used, exports) before designing
  yours. Caret components have a vocabulary; match it.
- **No match** — proceed to `pipeline-tokens.md`.

## What "matching" actually means

Match by lifecycle + shape, not by name.

- `<Spinner>` and `<UploadProgress>` are different shapes (one
  doesn't take a task list) but both `async-resolution`.
- `<Prompt.Text>` and `<Search>` are both `interactive` but the
  shapes differ — search has a result list, prompt has a single
  buffer.
- `<Banner>` and `<Splash>` are both `immediate-render` (or close to
  it) but the shapes differ — banner is one line, splash takes the
  whole viewport.

If the lifecycle differs, it's a different component. If the shape
differs, it might still be a customisation of the existing one.

## Registry catalogue (snapshot — verify with `caret-cli list`)

This list is informational; the live list wins.

**interactive**: prompt (text/password/confirm/select/multi-select/
number), spinner, splash, typewriter, reveal, boot, form, modal,
toast, autocomplete, editor, pager, search, tabs, accordion, toggle,
scrollable

**display**: banner, divider, key-value, table, progress, step-list,
log, error, panel, list, badge, tree, code-block, diff, kbd, tag,
breadcrumb, link, file-status

**utility**: paint (colour helpers), capability, motion, notify,
text-to-art, image-to-art, typography

If the live list returns something not on this snapshot, trust the
live list and **note that this rule is out of date** so it can be
refreshed.

## What to record before moving on

In your scratch buffer, write **either**:

- "No registry match — designing new" + link to closest adjacent
  component in `registry/components/`, or
- "Variant of `<X>` — customising via tokens" + name of the token,
  or
- "Already exists — recommended `caret-cli add <X>`, stopped."

Move to `pipeline-tokens.md` only if you wrote the first line.
