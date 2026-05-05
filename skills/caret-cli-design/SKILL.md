---
name: caret-cli-design
description: Read an existing CLI repo and produce a Caret adoption plan — which Caret components replace which existing surfaces, which tokens to customize, and the exact `npx caret-cli add` sequence to run. Use when the user wants to "make this CLI look like Caret" or "design this CLI properly," not when they are starting fresh (use `npx caret-cli init` for that).
---

# caret-cli-design

Audit a real CLI codebase and produce a **design plan** that brings
it inside the Caret system. Not a refactor — a plan. The user
decides which steps to apply; this skill's job is to turn a folder
of `console.log`s, ad-hoc `chalk` calls, and homegrown spinners
into a punch list of Caret components and token decisions.

> If the user wants to start a new CLI from scratch, do not invoke
> this skill — point them at `npx caret-cli init`. This skill is
> for **existing** repos with their own surface area.

## What this skill produces

A single artifact: a markdown plan with five sections:

1. **Inventory** — what surfaces the CLI has today (commands,
   prompts, errors, progress, logs, splash, help text, version)
2. **Mapping** — for each surface, the Caret component that replaces
   it and why (or "leave alone" with rationale)
3. **Token decisions** — accent, semantic colors, typography scale,
   symbol set; what to customize vs. take from defaults
4. **Adoption order** — a topological sort of `npx caret-cli add`
   commands so the user can land each replacement in isolation,
   with a working CLI between every step
5. **Risks** — places where the existing CLI's behavior conflicts
   with Caret's manifesto (e.g. uses the terminal bell, renders
   hex colors, ignores NO_COLOR), and what the user has to give up

The plan is not the patch. It tells the user *what to do*; running
the actual `caret-cli add` commands and rewriting files is up to
them (or a follow-up turn that explicitly asks for it).

## Pipeline — run in order

Each step is a procedural rule. Read each on demand.

1. **Inventory the codebase** — `rules/pipeline-inventory.md`
   Read package.json, the CLI entry point, README, and any
   commands/ folder. Identify every place the CLI prints to stdout
   or asks for input. Don't guess; read.
2. **Identify surfaces** — `rules/pipeline-surfaces.md`
   Group the inventory into Caret's interaction taxonomy: prompts,
   spinners, splash, errors, tables, progress, banners, key-values,
   step indicators, logs.
3. **Map to Caret components** — `rules/pipeline-mapping.md`
   For each surface, find the Caret component that fits. `caret-cli
   list` is the source of truth. If nothing fits, say so — the
   sister skill `create-caret-component` handles new components.
4. **Decide tokens** — `rules/pipeline-tokens.md`
   Pull the brand color from the README/logo; map it to
   `theme.color.accent`. Decide whether to override semantic
   defaults (rarely a good idea — the defaults are tuned).
5. **Order the adoption** — `rules/pipeline-adoption-order.md`
   Sort so that high-leverage / low-risk changes land first. The
   CLI must compile and run after every step.
6. **Surface risks** — `rules/pipeline-risks.md`
   Manifesto violations the existing code commits, and what the
   user must accept losing.
7. **Write the plan** — `rules/pipeline-write-plan.md`
   The deliverable. A single markdown document with the five
   sections listed above.

## Rules index

### Pipeline (orchestration)

- `pipeline-inventory` _(CRITICAL)_ — You cannot plan what you have not read. The single most common failure mode is recommending Caret components for surfaces that don't exist yet, or missing surfaces that do.
- `pipeline-surfaces` _(CRITICAL)_ — Without grouping, the mapping step balloons. Caret has a fixed taxonomy; learn it once, apply it to every CLI.
- `pipeline-mapping` _(HIGH)_ — The whole skill is justified by this step. Half-right mappings are worse than honest "no fit" — say so.
- `pipeline-tokens` _(HIGH)_ — Tokens are how brand identity survives the system. Get the accent right; default the rest.
- `pipeline-adoption-order` _(HIGH)_ — A plan that requires a 600-line PR is a plan the user will not execute. Order matters.
- `pipeline-risks` _(MEDIUM)_ — Surfaces what the user gives up. Some users will reject the plan if they need their custom hex palette; the plan must let them.
- `pipeline-write-plan` _(MEDIUM)_ — The deliverable. Procedural; the rule file enforces section structure.

### Reference tables

- `surface-taxonomy` _(HIGH)_ — Caret's official surface list. Use this to group; do not invent categories.
- `component-catalog` _(HIGH)_ — Which Caret components exist as of latest. Mirrors `caret-cli list`. If a referenced component is not here, the registry has changed — fall back to a live `caret-cli list` and update this rule.

### Validators (run before delivering the plan)

- `validate-runs-after-each-step` _(CRITICAL)_ — Every adoption step must leave the CLI in a working state. If step 3 requires step 5 to compile, reorder.
- `validate-no-orphan-mappings` _(HIGH)_ — Every surface in the inventory must appear in the mapping (with a Caret component, "create new", or "leave alone").
- `validate-honest-risks` _(HIGH)_ — If the existing code uses the terminal bell, the plan must say it has to be removed. Don't soften.

## Reference

- Caret manifesto: `https://caret.dev/spec`
- Live component catalog: `npx caret-cli list`
- Sister skill (per-component author): `create-caret-component`
- The user installs components with: `npx caret-cli add <name>`
