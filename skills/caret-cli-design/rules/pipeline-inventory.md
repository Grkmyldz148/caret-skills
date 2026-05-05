# pipeline-inventory

> First step. Read this when the user asks for a Caret adoption
> plan. Before you read any other rule.

You cannot recommend a Caret component for a surface that doesn't
exist, and you cannot ignore a surface that does. The single most
common failure mode of this skill is reading 30% of the codebase
and writing a plan as if it were the whole thing.

## What to read — in order

For any user CLI repo:

1. **`package.json`** — entry point (`bin`), dependencies
   (does it use chalk, ora, prompts, inquirer, listr, ink?),
   scripts, name, version. Tells you what shape of CLI it is.
2. **`README.md`** — the user-facing description. Brand voice,
   feature list, screenshots if any. The README is what users
   encounter first; the CLI's surfaces should match its tone.
3. **CLI entry point** — usually `bin/cli.js`,
   `src/index.ts`, or whatever `package.json` `bin` points to.
   Read it whole. Don't skim.
4. **Command handlers** — wherever the entry point dispatches.
   For commander/yargs/oclif setups, this is a `commands/` dir.
   For ad-hoc CLIs, it's all in the entry point.
5. **Output helpers** — any file that wraps `console.log` /
   `process.stdout.write`. These are where the design choices
   currently live.
6. **Tests / snapshot files** — sometimes the only place the
   actual rendered output appears. Worth a 60-second skim.

If the repo has 50+ files, you can stop after the entry point and
the largest command handler — that usually covers the surfaces.
But **read the README in full** every time; brand voice is rarely
documented elsewhere.

## What to record

For every place the CLI prints to stdout or asks for input, write
one line in your scratch buffer:

```
<file>:<line> — <one-line description of what it does>
```

Example for a fictional `acme-cli`:

```
src/index.ts:14   — banner ASCII art on first run
src/index.ts:22   — chalk.bold.cyan splash with version
src/commands/init.ts:8   — inquirer prompt for project name
src/commands/init.ts:23  — ora spinner during scaffolding
src/commands/init.ts:48  — chalk.green ✓ + summary list
src/commands/deploy.ts:12 — chalk.yellow warn before deploying
src/commands/deploy.ts:31 — listr task tree for deploy steps
src/commands/deploy.ts:67 — chalk.red error block + stack trace
src/lib/log.ts:5  — wraps console.log with timestamp
```

This list is the inventory. The next pipeline step (`pipeline-
surfaces.md`) groups it.

## What to skip

- Test snapshots that don't reflect runtime output.
- Generated files (`dist/`, `build/`, `.next/`).
- Internal telemetry code that doesn't render to the user.
- Logging in service of debugging (`if (DEBUG) console.log...`)
  unless the CLI ships in debug mode by default.

## What to flag if you can't read it

If the CLI is published to npm but you don't have the source
locally:

```bash
npm pack <package-name>
tar -xzf <package-name>-*.tgz
```

The unpacked tarball has the runtime files. The `dist/` is what
ships; read that. (Source maps, if shipped, point back to the
original TypeScript — follow them if possible.)

If the package ships compiled output only and the source is on
GitHub, clone the repo. Don't plan from compiled output unless
you have to — the surfaces are obscured.

## What to record before moving on

- The inventory list (file:line — description).
- The brand voice in one sentence (e.g. "Vercel-style: confident
  lowercase, technical, terse" or "Linear-style: minimal, no
  emoji, monospace-aware").
- The CLI's primary domain (deployment? scaffolding? data
  transformation? package management?).

Move to `pipeline-surfaces.md`.
