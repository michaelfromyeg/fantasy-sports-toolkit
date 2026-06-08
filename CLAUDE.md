# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This isn't an application with its own build. It's a
[Weft](https://github.com/michaelfromyeg/weft) marketplace. The deliverable is a
set of declarative plugin definitions (Markdown + YAML + JSON) that the external
`weft` CLI compiles into native plugin formats for five coding-agent harnesses
(Claude Code, Codex, Cursor, Copilot, OpenCode). There's no compiled source here;
you author content, and Weft is the toolchain.

`out/` and `.claude/` are Weft-generated and gitignored. Never hand-edit them.

## Commands

The toolchain is the external `weft` binary (`npm i -g @michaelfromyeg/weft-cli`,
or `npx @michaelfromyeg/weft-cli`). Run from the repo root, where `marketplace.yaml`
lives:

```sh
weft validate plugins/fantasy-manager   # static-validate one plugin before building
weft build . --out out                  # compile all plugins -> out/{claude,codex,cursor,copilot,opencode}
weft build . --out out --target claude  # a subset of harnesses
weft install . --scope project          # place compiled plugins into ./.claude (+ detected harnesses)
weft eval plugins/fantasy-manager       # run every skill's eval suite (claude harness)
```

`weft build`'s default out dir is `.weft-out`, but this repo standardizes on `out/`,
so always pass `--out out`. There's no separate lint/test step beyond `weft validate`
(schema/structure) and `weft eval` (behavioral evals); run both before declaring a
change done.

## Architecture

The organizing principle is a hard separation between judgment (sport- and
provider-agnostic logic) and data (a swappable provider). Hold this line when
editing; it's the whole point of the repo.

- `marketplace.yaml` is the catalog. It lists the plugins, the `dev.fantasylab`
  namespace, and `allow_dependencies_on`.
- `plugins/fantasy-manager` (`weft.yaml`, v0.3.x) is the agnostic brains. Each
  component is declared in its `weft.yaml` and lives in a conventional dir:
  - `skills/<name>/SKILL.md` holds auto-triggered skills (`set-lineup`,
    `waiver-wire`, `trade-analyzer`). The frontmatter `description` is what triggers
    the skill.
  - `commands/*.md` holds explicit fast-path commands (`/start-sit`, `/trade-check`).
  - `agents/*.md` holds the `waiver-scout` agent (scouts only; never submits claims).
  - `mcp/<name>/server.json` holds bundled provider-independent context MCPs
    (`sports-data`, `reddit`, `web-fetch`). These point at third-party servers, not
    code maintained here.
  - `evals/*.cases.yaml` holds per-skill case suites; see the eval format below.
- `plugins/sleeper` and `plugins/yahoo` (each a `weft.yaml`) are interchangeable MCP
  data providers, and the swap point: Sleeper (read-only, NFL+NBA, no key) and Yahoo
  (read+write across NFL/NHL/NBA/MLB via the `mcp-yahoo-fantasy` package run with
  `uvx`, one-time OAuth). The skills treat whichever is connected as a plain data
  source, so a future ESPN provider would slot in the same way and the skills work
  unchanged.

### Why skills must stay agnostic

Skills read the league's own roster slots and scoring at runtime rather than
hardcoding a sport, so the same `set-lineup` handles football, basketball, baseball,
and hockey. They treat any connected fantasy MCP as a pure data source and degrade
gracefully (ask the user to paste their roster) when no provider is connected. When
editing a skill, never bake in a specific sport, scoring system, or provider; push
that into the league data the skill reads.

### Eval format (evals/*.cases.yaml)

Each file targets one component (`component:`, `harnesses:`) and lists `cases`. A
case has a `prompt`, optional `setup`/`cleanup` shell (e.g. writing a `roster.md`
fixture), and `assert` entries of three kinds: `trace` (a tool like `Read` was
called, with `minPassRate`), `output` (regex `matches`), and `judge` (an LLM rubric).
Cover new sports and behaviors with all three kinds where it makes sense.

## Conventions

Versions are per-plugin in each `weft.yaml` (and mirrored in the MCP `server.json`s);
the npm package version in `package.json` is the marketplace's own release. The MCP
servers referenced are third-party, so this repo ships only `server.json` pointers.
Don't treat them as in-repo code to modify.
