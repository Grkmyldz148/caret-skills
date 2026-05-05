---
name: create-caret-component
description: Author a single Caret component (Ink/React for the terminal) that obeys the Caret manifesto — semantic colors, NO_COLOR awareness, capability fallbacks, no terminal bell, no custom symbols. Use when the user asks for a brand-new component or a custom variant that the registry does not yet ship.
---

# create-caret-component

Design **one** Caret-compliant component end-to-end: spec, props,
render, capability fallbacks, and a passing test. This is the skill
to invoke when the user asks for "a new <thing>" — not when they
want to install an existing one.

> If the user wants a component that already ships in the registry
> (`prompt`, `spinner`, `splash`, `modal`, `toast`, `form`, …),
> tell them to run `npx caret-cli add <name>` instead. Do not
> re-author what `caret list` already returns.

## What "Caret-compliant" means — read this first

Caret is a design system for CLIs. Components are not free-form Ink
JSX — they obey a manifesto. Skipping any of these turns your output
into "yet another chalk + ora demo," which is the failure mode this
skill exists to prevent.

1. **Semantic colors only.** Use `theme.color.{accent,success,warning,error,info,muted,subtle,fg,canvas}` — never hex literals, never `chalk.red`, never `'#5882f7'`. The user's terminal palette wins.
2. **NO_COLOR is gospel.** When `process.env.NO_COLOR` is set, every color call must degrade to plain text. Build on `paint()` from `@caret-cli/registry/lib/paint` — it already handles this.
3. **Capability fallbacks are mandatory.** Truecolor → 256 → 16 → mono. Unicode → ASCII. Animation → static. Never assume the user has a modern emoji-rendering terminal. Use `capability` from `@caret-cli/registry/lib/capability` to branch.
4. **Symbols come from `tokens.symbols` — never invent your own.** The cursor is `▍`. The success glyph is `✓`. Customising these breaks visual coherence with the rest of the system. Pick from the existing set; if the existing set lacks what you need, propose adding to `registry/tokens/symbols.ts`, do not inline an alternative in your component.
5. **No terminal bell, ever.** No `\x07`, no ``, no `process.stdout.write('\x07')`. Caret is silent.
6. **Motion respects `prefers-reduced-motion`.** Caret reads it from `process.env.CARET_REDUCE_MOTION` and from the OS where available. Use `motion` helpers from `registry/lib/motion`.
7. **Pipes get plain text.** When stdout is not a TTY, render the simplest possible inline output — no boxes, no spinners, no live regions.
8. **Brand accent is fixed truecolor.** It is the one exception to the "user's palette wins" rule, because it is brand identity. `theme.color.accent` resolves to truecolor when supported, falls back to bright cyan otherwise — never override.

These rules are enforced by `validate-manifesto.md`. Run that
validator before considering the component done.

## Pipeline — run in order

Each step is a procedural rule. Open the rule file with `Read` the
moment you start the step.

1. **Inventory the request** — `rules/pipeline-inventory.md`
   What is the component for? What inputs does it take? What is its
   resolution lifecycle (immediate / async / interactive)? Does it
   render once, animate, or stay live?
2. **Check the registry** — `rules/pipeline-registry-check.md`
   Run `caret-cli list`. If a similar component exists, base on it
   (cite the file path). Do not duplicate.
3. **Choose tokens** — `rules/pipeline-tokens.md`
   Pick the symbol(s), the semantic color(s), and the spacing /
   typography scale. Tokens are the contract; everything else is
   implementation detail.
4. **Define props + lifecycle** — `rules/pipeline-api.md`
   Props schema, default values, error states, async resolution.
   Aim for fewer props with sensible defaults.
5. **Write the render** — `rules/pipeline-render.md`
   Build the JSX tree. Branch on capability for any glyph or color
   that has a non-trivial fallback. Keep one render path per
   capability tier.
6. **Add a test** — `rules/pipeline-test.md`
   `ink-testing-library` for interactive components, snapshot for
   pure-render components. The test must cover the NO_COLOR path
   and the no-TTY path at minimum.
7. **Validate** — `rules/validate-*.md`
   All validators in this folder must pass. Each rule file has
   thresholds; do not summarize from memory.

## Rules index

### Pipeline (run in order)

- `pipeline-inventory` _(CRITICAL)_ — Without a clear lifecycle, the API drifts. Two minutes here saves an hour of redesign.
- `pipeline-registry-check` _(CRITICAL)_ — Re-implementing an existing Caret component is the most common failure mode. Always check first.
- `pipeline-tokens` _(CRITICAL)_ — Tokens are the difference between "Caret component" and "yet another Ink box". Skipping this guarantees a manifesto violation.
- `pipeline-api` _(HIGH)_ — Bad props compound. Fix the API before you write 200 lines of render that depend on it.
- `pipeline-render` _(HIGH)_ — Capability branching at this layer is where reduced-color terminals get respected.
- `pipeline-test` _(MEDIUM)_ — A failing test catches the manifesto violations that a human reviewer would miss.

### Validators (run after the file is written)

- `validate-manifesto` _(CRITICAL)_ — Catches hex colors, terminal bell, custom symbols, ignored NO_COLOR. This is the floor.
- `validate-capability` _(HIGH)_ — Components that look great on the author's iTerm and break in dumb terminals are the second most common failure mode.
- `validate-no-bell` _(CRITICAL)_ — Worth a dedicated check; one stray `\x07` ships and the manifesto reads as a lie.
- `validate-symbol-source` _(HIGH)_ — Inline glyphs are the silent killer. Force every symbol through `tokens.symbols`.

## Reference

- Caret manifesto: `https://caret.dev/spec`
- Registry source: `npm view caret-cli` → unpacks to `registry/`
- Sister skill (whole-CLI pass): `caret-cli-design`
