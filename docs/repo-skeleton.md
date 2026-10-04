# Repo skeleton

All four vendor repos were scaffolded from the same template.
Keeping the skeleton identical means a fix in one repo can be copied to the others file for file.

## File list

```
<vendor>-axi/
  bin/<vendor>-axi.ts          entrypoint, --version fast path
  src/cli.ts                   runAxiCli, help text, formatError hook
  src/version.ts               leaf module, node builtins only
  src/<cli>.ts                 the only module that spawns the vendor binary
  src/errors.ts                AxiError, ordered error patterns
  src/args.ts                  flag parsing helpers, rejection of leftovers
  src/toon.ts                  TOON rendering helpers
  src/commands/home.ts         no-args dashboard
  src/commands/<command>.ts    one file per top-level command
  test/*.test.ts               offline Vitest tests
  skills/<vendor>-axi/SKILL.md the shipped agent skill
  AGENTS.md                    agent memory for this repo
  CLAUDE.md                    one line: @AGENTS.md
  CONTRIBUTING.md              how to raise a PR (no-mistakes), release rules
  VISION.md                    scope, interface, safety rules
  README.md
  LICENSE                      MIT
  .github/workflows/ci.yml
  .github/workflows/release-please.yml
  .github/workflows/guard-generated-files.yml
  release-please-config.json, .release-please-manifest.json
  .prettierignore
  .gitignore
  package.json, pnpm-lock.yaml, tsconfig.json
```

[railway-axi](https://github.com/simkimsia/railway-axi) is the most complete example of this list.
Gaps per repo are tracked in [conformance.md](conformance.md).

## package.json

Copy [railway-axi/package.json](https://github.com/simkimsia/railway-axi/blob/main/package.json) and rename.
The fields that matter:

- `"name": "@simkimsia/<vendor>-axi"`, a scoped name nobody else can own ([distribution.md](distribution.md)).
- `"type": "module"`, because `axi-sdk-js` and TOON are ESM.
- `"bin": { "<vendor>-axi": "./dist/bin/<vendor>-axi.js" }`, so the published package runs compiled JS and never needs `tsx`.
- `"files": ["dist", "skills/<vendor>-axi", "LICENSE", "README.md"]`, so the skill ships inside the npm package and tests do not.
- `"packageManager": "pnpm@10.33.0"`, which CI reads to pick the pnpm version.
- `"engines": { "node": ">=20" }`, the floor `axi-sdk-js` supports.
- Scripts: `build`, `test`, `test:watch`, `dev`, `format`, `format:check`, `prepublishOnly`. `prepublishOnly` builds so a publish can never ship a stale `dist/`.
- Dependencies: `axi-sdk-js` (^0.1.11) and `@toon-format/toon` (^2.1.0). Nothing else at runtime.
- Dev dependencies: `typescript`, `vitest` ^3, `prettier`, `tsx`, `@types/node`.
- `"repository"` pointing at the GitHub repo. The SDK's built-in `update` command resolves the npm name from this package, so the name must be one you own.

## tsconfig.json

Copy [railway-axi/tsconfig.json](https://github.com/simkimsia/railway-axi/blob/main/tsconfig.json) unchanged.
It uses `Node16` module resolution, so import specifiers end in `.js`.
It compiles `src/` and `bin/` only and excludes `test/`, which Vitest runs from source.

## CI

[railway-axi/.github/workflows/ci.yml](https://github.com/simkimsia/railway-axi/blob/main/.github/workflows/ci.yml) runs install, build, `format:check`, and test on every push to main and every PR.
Build runs before test so a type error fails fast, and `format:check` keeps diffs reviewable.
It runs on Node 24, the same as gh-axi, while `engines` stays at `>=20`.
A copy is in [templates/.github/workflows/ci.yml](../templates/.github/workflows/ci.yml), and the file is byte-identical in all four repos.
The template also carries a `paths-ignore` block for release-please's generated files, which reaches each repo with the release rollout ([distribution.md](distribution.md#releases-release-please-and-trusted-publishing)).

CI is required, not optional.
Without any CI check on a PR, the no-mistakes CI step never reports ready, so the gate stalls ([process.md](process.md#the-no-mistakes-gate)).
I learned this on 2026-10-04.

## .prettierignore

Two lines: `pnpm-lock.yaml` and `CHANGELOG.md`.
Both are generated, so Prettier has no business checking them.
A pnpm upgrade that changes the lockfile layout should not fail `format:check`.
release-please writes `CHANGELOG.md` with `*` bullets and extra blank lines, which Prettier rewrites; without the ignore, the first release turns `main` red, and the release guard stops anyone from hand-fixing the file.
See [railway-axi/.prettierignore](https://github.com/simkimsia/railway-axi/blob/main/.prettierignore).

## LICENSE

MIT, `Copyright (c) 2026 KimSia Sim`, the same text in every repo.
MIT matches gh-axi and `axi-sdk-js`, which keeps contributions simple.

## AGENTS.md and CLAUDE.md

`AGENTS.md` is the agent memory for the repo, with fixed sections: What this is, Architecture, `<Vendor>` CLI notes, Conventions, Maintaining this file.
`CLAUDE.md` contains only `@AGENTS.md`, so Claude Code and other agents read one file.

The CLI notes section is the most valuable part.
It records verified quirks of the vendor CLI (output shapes, exit codes that lie, commands that hang), with the version checked.
See [cloudflare-axi/AGENTS.md](https://github.com/simkimsia/cloudflare-axi/blob/main/AGENTS.md) for a dense example.

## CONTRIBUTING.md

`CONTRIBUTING.md` is for people, `AGENTS.md` is for agents.
It is adapted from [gh-axi's](https://github.com/kunchenguid/gh-axi/blob/main/CONTRIBUTING.md): raise PRs through no-mistakes, use conventional commits, never hand-edit release-please output, and keep install steps in the README only.
The release guard's error message points contributors at it, so every repo with `guard-generated-files.yml` needs one.
Template: [templates/CONTRIBUTING.md](../templates/CONTRIBUTING.md).

## Using the templates

Copy [`templates/`](../templates) into the new repo root, rename `skills/vendor-axi/` to `skills/<vendor>-axi/`, and replace these placeholders:

| Placeholder | Meaning | Railway example |
| --- | --- | --- |
| `<vendor>` | lowercase vendor name, used in the package name | `railway` |
| `<Vendor>` | display name | `Railway` |
| `<cli>` | the wrapped binary | `railway` |
| `<VENDOR>` | uppercase prefix for error codes | `RAILWAY` |
| `<surfaces>` | the areas the axi covers | projects, services, deployments, logs |

Any other `<...>` text in the templates, such as `<version>` or a bracketed sentence, is a prompt to fill in or delete.

Source files are not templated.
Copy `bin/`, `src/version.ts`, `src/toon.ts`, `src/args.ts`, `src/errors.ts`, and `src/cli.ts` from railway-axi and rename, because they are already generic and tested.
