# axi-playbook

How I build a `<vendor>-axi`, and why each piece is the way it is.

[AXI](https://github.com/kunchenguid/axi) is Kun Chen's spec for agent-ergonomic CLIs: token-efficient [TOON](https://toonformat.dev) output, structured errors, definitive empty states, and help that tells the agent what to run next.
The reference implementation is [gh-axi](https://github.com/kunchenguid/gh-axi).

I maintain four vendor wrappers built on the same template:

- [railway-axi](https://github.com/simkimsia/railway-axi) wraps the Railway CLI.
- [cloudflare-axi](https://github.com/simkimsia/cloudflare-axi) wraps `wrangler` plus a few Cloudflare REST endpoints.
- [netlify-axi](https://github.com/simkimsia/netlify-axi) wraps the Netlify CLI.
- [calcom-axi](https://github.com/simkimsia/calcom-axi) wraps the Cal.com CLI.

This repo writes down the conventions they share, so the fifth one starts consistent and the first four can be pulled back into line.
It also covers how I use axis I did not write, such as gh-axi.

## Who this is for

- Me, before starting a new wrapper or a large change to an existing one.
- Coding agents working in any of the four repos.
- Anyone building their own `xxx-axi` who wants a worked set of conventions to copy or argue with.

The 10 AXI principles are not restated here.
Read them in the upstream skill, [SKILL.md](https://github.com/kunchenguid/axi/blob/main/.agents/skills/axi/SKILL.md) (§1 to §10), with one-line summaries in [principles.yaml](https://github.com/kunchenguid/axi/blob/main/principles.yaml).
This playbook covers the decisions the spec leaves open.

## Docs

| Doc | What it covers |
| --- | --- |
| [why.md](docs/why.md) | Why wrap a CLI at all, and why gh-axi is the model |
| [repo-skeleton.md](docs/repo-skeleton.md) | File list, package.json, tsconfig, CI, license, using the templates |
| [architecture.md](docs/architecture.md) | Fast path, the single spawner, error mapping and codes, args, TOON, help |
| [backend-policy.md](docs/backend-policy.md) | CLI first, when REST or GraphQL is allowed, auth reuse, MCP stance |
| [conditional-modules.md](docs/conditional-modules.md) | Secret redaction, ambiguity refusal, write safety, session hooks |
| [testing.md](docs/testing.md) | Offline tests, verbatim stderr fixtures, live smoke for writes |
| [process.md](docs/process.md) | Branches, the no-mistakes gate, merges, commits, gap issues |
| [distribution.md](docs/distribution.md) | npm naming, install, publish gotchas, release-please and trusted publishing |
| [ecosystem.md](docs/ecosystem.md) | Agent skill, catalog entry, scoring, benchmarks, VISION.md and triage |
| [using-others-axis.md](docs/using-others-axis.md) | The axi-first rule, gap and bypass handling, writing upstream issues |
| [conformance.md](docs/conformance.md) | Which repo follows which convention, and the drift to fix |

## Templates

[`templates/`](templates) holds copyable starter files: `VISION.md`, `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, the agent skill, the CI and release workflows, the release-please config and manifest, `.gitignore`, and `.prettierignore`.
Placeholders are listed in [repo-skeleton.md](docs/repo-skeleton.md#using-the-templates).

## Conformance at a glance

The full matrix is in [conformance.md](docs/conformance.md).
The short version:

- **Shared by all four:** the scaffold (bin, tsconfig, package.json), the `--version` fast path, one spawner module, ordered error patterns, `formatError` hook, args rejection, the shipped agent skill, and a catalog entry upstream.
- **Published:** all four are on npm as `@simkimsia/<vendor>-axi@0.1.0` since 2026-10-04.
- **Present in some:** VISION.md (railway, cloudflare), triage crewmate (cloudflare), truncation with `--full` (calcom).
- **Merged in all four on 2026-10-04:** CI on Node 24, `.prettierignore`, the npm-first README install section, and the skill Setup pointer to the README.
- **Rolling out via PRs:** release automation with release-please and npm trusted publishing, starting with railway-axi.
- **Present in none yet:** session hooks, `bench/`.
- **Known drift:** the error code for "no linked context" and unwritten backend stances for netlify and calcom.

## License

[MIT](LICENSE)
