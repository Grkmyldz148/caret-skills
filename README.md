# caret-skills

Skills for **Caret** — the design system for modern command-line
tools. Drop these into Claude Code (or any agent runtime that
supports skills) and the model gains structured, manifesto-aware
expertise on authoring Caret components and adopting Caret in
existing CLIs.

## Bundled skills

- **`create-caret-component`** — Author one Caret-compliant component
  end-to-end: tokens, props, render, capability fallbacks, test.
  Manifesto-checked. See [`skills/create-caret-component/SKILL.md`](skills/create-caret-component/SKILL.md).
- **`caret-cli-design`** — Read an existing CLI repo and produce a
  Caret adoption plan: which components replace which surfaces,
  which tokens to customize, the exact `npx caret-cli add` order,
  and the manifesto risks. See [`skills/caret-cli-design/SKILL.md`](skills/caret-cli-design/SKILL.md).

These pair with the runtime CLI:

```bash
npx caret-cli@alpha init my-cli      # scaffold a new CLI
npx caret-cli@alpha list             # browse the component catalog
npx caret-cli@alpha add prompt       # copy a component into your project
```

## Install — Claude Code (native plugin)

```
/plugin marketplace add Grkmyldz148/caret-skills
/plugin install caret-skills@caret-skills
```

Lands in `~/.claude/plugins/`. Both skills become invokable as
`/create-caret-component` and `/caret-cli-design` immediately.
Future skills added here arrive via `/plugin update caret-skills`.

## Install — any agent runtime (`npx skills`)

```bash
npx skills add Grkmyldz148/caret-skills
```

Uses the [`skills`](https://www.npmjs.com/package/skills) CLI
(vercel-labs). Picks the runtime interactively (Claude Code, Cursor,
Codex, OpenCode, 50+ supported), shows a multi-select picker so you
can install both skills in one go or just the one you need.

## When to use which

| You want… | Skill to invoke |
| --- | --- |
| To start a new CLI from a template | _none_ — run `npx caret-cli@alpha init` directly |
| To install a component that already exists in the registry | _none_ — run `npx caret-cli@alpha add <name>` directly |
| To author a brand-new component the registry doesn't ship | `create-caret-component` |
| To plan how to bring an existing CLI inside the Caret system | `caret-cli-design` |

## Updating

PRs welcome on this repo. The skills are first-party — no auto-mirror
yet, so what's committed here is what ships.

---

Caret is in active design. Both the runtime CLI and these skills are
versioned `0.x` and may break across releases.
