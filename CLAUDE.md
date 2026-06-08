# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **not** an application with its own build — it is a [Weft](https://github.com/michaelfromyeg/weft)
marketplace. The deliverable is a set of declarative plugin definitions (Markdown
+ YAML + JSON) that the external `weft` CLI compiles into native plugin formats
for five coding-agent harnesses (Claude Code, Codex, Cursor, Copilot, OpenCode).
There is no compiled source in this repo; you author content, and Weft is the toolchain.

`out/` and `.claude/` are Weft-generated and gitignored — never hand-edit them.

## Commands

The toolchain is the external `weft` binary (`npm i -g @michaelfromyeg/weft-cli`,
or `npx @michaelfromyeg/weft-cli`). Run from the repo root (where `marketplace.yaml` lives):

```sh
weft validate plugins/fantasy-manager   # static-validate one plugin before building
weft build . --out out                  # compile all plugins -> out/{claude,codex,cursor,copilot,opencode}
weft build . --out out --target claude  # build a subset of harnesses
weft install . --scope project          # place compiled plugins into ./.claude (+ detected harnesses)
weft eval plugins/fantasy-manager       # run every skill's eval suite (claude harness)
```

Note: `weft build`'s default out dir is `.weft-out`, but this repo standardizes on
`out/` — always pass `--out out`.

There is no separate lint/test step beyond `weft validate` (schema/structure) and
`weft eval` (behavioral evals). Run both before declaring a change done.

## Architecture

The organizing principle is a hard separation between **judgment** (sport- and
provider-agnostic logic) and **data** (a swappable provider). Hold this line when
editing — it is the whole point of the repo.

- **`marketplace.yaml`** — the catalog. Lists the plugins, the `dev.fantasylab`
  namespace, and `allow_dependencies_on`.
- **`plugins/fantasy-manager`** (`weft.yaml`, v0.3.x) — the agnostic brains. Each
  component is declared in its `weft.yaml` and lives in a conventional dir:
  - `skills/<name>/SKILL.md` — auto-triggered skills (`set-lineup`, `waiver-wire`,
    `trade-analyzer`). Frontmatter `description` is what triggers the skill.
  - `commands/*.md` — explicit fast-path commands (`/start-sit`, `/trade-check`).
  - `agents/*.md` — the `waiver-scout` agent (scouts only; never submits claims).
  - `mcp/<name>/server.json` — bundled *provider-independent* context MCPs
    (`sports-data`, `reddit`, `web-fetch`). These are pointers to third-party
    servers, not code maintained here.
  - `evals/*.cases.yaml` — per-skill case suites; see eval format below.
- **`plugins/sleeper`** (`weft.yaml`, v1.1.x) — a single MCP **data provider**
  (read-only Sleeper API, NFL+NBA, no key). This is the swap point: a future
  `espn`/`yahoo` provider would slot in the same way and the skills work unchanged.

### Why skills must stay agnostic

Skills read the league's *own* roster slots and scoring at runtime rather than
hardcoding a sport, so the same `set-lineup` handles football/basketball/baseball/
hockey. They treat any connected fantasy MCP as a pure data source and degrade
gracefully (ask the user to paste their roster) when no provider is connected.
When editing a skill, never bake in a specific sport, scoring system, or provider —
push that into the league data the skill reads.

### Eval format (`evals/*.cases.yaml`)

Each file targets one component (`component:`, `harnesses:`) and lists `cases`.
A case has a `prompt`, optional `setup`/`cleanup` shell (e.g. writing a `roster.md`
fixture), and `assert` entries of three kinds: `trace` (a tool like `Read` was
called, with `minPassRate`), `output` (regex `matches`), and `judge` (an LLM
rubric). Cover new sports/behaviors with all three kinds where it makes sense.

## Conventions

- Versions are per-plugin in each `weft.yaml` (and mirrored in MCP `server.json`s);
  the npm package version in `package.json` is the marketplace's own release.
- The MCP servers referenced are third-party — this repo ships only `server.json`
  pointers. Don't treat them as in-repo code to modify.
