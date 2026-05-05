# pipeline-write-plan

> Final pipeline step. The deliverable.

Everything you've written into the scratch buffer in earlier
steps now goes into a single markdown document with a fixed
structure. The structure is fixed because the user will skim, not
read; predictable headings reward skimming.

## The required structure

```markdown
# Caret adoption plan — <CLI name>

## 1. Inventory
[Inventory table from pipeline-surfaces.md, after grouping]

## 2. Mapping
[Mapping table from pipeline-mapping.md]

## 3. Token decisions
[Token decisions block from pipeline-tokens.md]

## 4. Adoption order
[Step-by-step plan from pipeline-adoption-order.md]

## 5. Risks
[Risk table from pipeline-risks.md]

---

## Appendix — about Caret
- Manifesto: https://caret.dev/spec
- Components: `npx caret-cli list`
- Install runtime: `npx caret-cli init` (new project)
  or `npx caret-cli add <component>` (existing project)
```

Five sections, one appendix. Section count is fixed; section
contents are what you produced earlier.

## Style notes

- **Lead with the structure, not prose.** The user skims.
  Long paragraphs get skipped; tables and bullets get read.
- **Be specific.** "Replace logging" is not a step; "Replace
  `console.log` calls in `src/index.ts:5,12,28` with
  `log.info`" is.
- **Quote the user's existing code.** Show the before/after
  for the most complex one or two surfaces. Builds confidence
  the plan understood the codebase.
- **No emoji.** The plan is technical. Caret doesn't ship them
  in components; the plan shouldn't either.

## Length target

For a small CLI (1-3 commands, < 20 surfaces): 1-2 screens.
For a medium CLI (5-10 commands, 20-60 surfaces): 3-5 screens.
For a large CLI (oclif-style, 50+ commands): split into
multiple plans (one per command group). Never a single plan
longer than 5 screens — it stops being read.

## What to ask the user before publishing the plan

- **"Do you want a follow-up turn that applies these
  changes?"** — Some users want the plan; others want a
  pull request. Both are fine; ask which.
- **"Are any of the manifesto-blockers in section 5
  unacceptable?"** — If the user says yes, the plan is moot;
  go back to `pipeline-risks.md` "if a manifesto-blocker is
  unacceptable" and write the alternative recommendation.

Don't ask these questions inside the plan document. Ask them
after delivering it.

## Final pre-delivery check

Before sending the plan, run the three validators in this order:

1. `validate-runs-after-each-step.md` — every adoption step
   leaves a working CLI?
2. `validate-no-orphan-mappings.md` — every inventory surface
   appears in the mapping table?
3. `validate-honest-risks.md` — risks are not softened?

Each is short. If any fails, fix the plan, don't ship it.

## What to record

Nothing — the plan is the artifact. After delivery, the skill is
done.
