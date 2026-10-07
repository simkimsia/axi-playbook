# Using other people's axis

I use axis I did not write every day, mostly kunchenguid's [gh-axi](https://github.com/kunchenguid/gh-axi).
The rules below apply to every axi I have installed, mine or not.

## Axi first

When an axi is installed for a vendor, use it instead of the plain CLI: `gh-axi` over `gh`, `railway-axi` over `railway`, `cloudflare-axi` over `wrangler`.

Fall back to the plain CLI only when:

- the axi is not installed (`command -v <name>-axi` fails), or
- the operation you need is not wrapped.

Why: an instruction in an agent's memory file is a suggestion, and agents drift back to the CLI they saw most in training.
Using the axi by default is also how gaps get found. A silent fallback hides the gap forever.

## When an axi I maintain has a gap

1. File an issue on the axi's repo: the plain command that works, what the axi did, and the axi command shape you expected.
2. Rerun the plain command prefixed with the issue reference: `AXI_GAP=simkimsia/<vendor>-axi#<n> railway domain list`.
3. At the end of the task, link the issue and offer to fix it.

The prefix ties every fallback to an open issue, so gaps become a backlog.

## When someone else's axi has a gap

1. Rerun the plain command prefixed `AXI_BYPASS=1`.
2. Name the unwrapped command in your reply.
3. Ask before filing upstream.

Why ask: an issue on someone else's repo is a request for their time.
It should be one the maintainer wants, written once, and checked against their open issues first.

## Enforcement: axi-enforce

[axi-enforce](https://github.com/simkimsia/axi-enforce) puts a check in front of every shell command an agent runs, using each agent's own hook system (Claude Code, Codex, OpenCode, Gemini CLI, Qwen Code, Cursor).

- A plain CLI call is refused when its axi is installed, with a message pointing at the axi's `--help`.
- `AXI_GAP=owner/repo#N` is allowed only if that issue exists and is open.
- `AXI_BYPASS=1` is allowed for axis you do not maintain.
- Every allowed bypass is logged, and `axi-enforce log` shows them.

The bypass log is the input for the next round of work on my own axis.

## Writing a good upstream issue

Structure:

- **Problem**: what the agent saw, with the exact command and the output, and why it matters to an agent.
- **Proposed fix**: the smallest change that solves it, with the source file and line that cause it.
- **Environment**: axi version, wrapped CLI version, OS.

Before filing:

- Reproduce with the plain CLI, using the exact argv the axi forwarded (`AXI_DEBUG=1` where the axi supports it, see [architecture.md](architecture.md#one-module-spawns-the-vendor-binary)). Retyping the query by hand tests your translation, not the axi's.
- If the plain CLI fails the same way, still file it. It is an inherited gap, and the axi still owes a workaround, a mapped error, or a documented limit ([process.md](process.md#inherited-gaps)).
- Read the wrapper's source and cite the line. A maintainer can confirm a cited cause in a minute.
- Search open and closed issues.

Examples on gh-axi:

- [#180](https://github.com/kunchenguid/gh-axi/issues/180): `api graphql --field` returns the schema because `--method GET` is always forwarded. Cites the cause in `src/commands/api.ts`.
- [#181](https://github.com/kunchenguid/gh-axi/issues/181): wrap `gh discussion`. States which `gh` subcommands exist, and the real task that needed them.
- [#134](https://github.com/kunchenguid/gh-axi/issues/134): publish an existing local repo (`gh repo create --source --push`). Fixed by merged PRs [#135](https://github.com/kunchenguid/gh-axi/pull/135) and [#137](https://github.com/kunchenguid/gh-axi/pull/137), both raised through no-mistakes.

A counter-example, kept for the lesson: [#163](https://github.com/kunchenguid/gh-axi/issues/163) (`search issues` with a `repo:` qualifier plus more terms) was closed as "native `gh` fails too, so not a wrapper defect".
The plain-CLI repro passed the whole query as one string, which is what gh-axi itself was doing wrong. gh-axi joined the terms, and `gh` quotes a multi-word argument that starts with a qualifier.
[#188](https://github.com/kunchenguid/gh-axi/pull/188) fixes it in the wrapper by forwarding each term as its own argument.

When the maintainer agrees, offer the PR yourself, following the upstream repo's contribution rules ([process.md](process.md#the-no-mistakes-gate)).
