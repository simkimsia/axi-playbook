# Process

## Branch per change

Every change starts on a feature branch off `main`.
`main` only moves by merged PR, so its history is a list of reviewed changes.

## The no-mistakes gate

[no-mistakes](https://github.com/kunchenguid/no-mistakes) puts a local git proxy in front of the real remote.
`git push no-mistakes` runs a pipeline (review, tests, lint, docs) on the branch, applies or proposes fixes, then pushes and opens the PR.

Why: an automated reviewer catches the class of bug a solo maintainer misses on their own diff.
On railway-axi it caught several real secret-leak paths in `variables set` before merge (see the `no-mistakes(review): ...` commits in the [railway-axi history](https://github.com/simkimsia/railway-axi/commits/main)).

Status today: railway-axi uses the gate. cloudflare-axi PRs were opened without it. netlify-axi and calcom-axi have had no PRs yet. New work in all four should go through it.

Upstream repos by kunchenguid (axi, gh-axi) require it for human-authored PRs, and a check fails a PR whose body lacks the no-mistakes signature.
See upstream [CONTRIBUTING.md](https://github.com/kunchenguid/axi/blob/main/CONTRIBUTING.md). Fork contributions need no-mistakes v1.30.1 or newer.

One quirk: the pipeline rewrites the PR body on each run, so re-add any `Closes #N` line after every run.

The gate needs a CI workflow in the repo.
Its CI step waits for the PR's checks, and when a repo has no CI check at all, that step never reports ready.
I learned this on 2026-10-04, which is why every repo carries [the CI workflow](repo-skeleton.md#ci).

## Rebase-merge

PRs merge with rebase-merge, so each commit lands on `main` as written.

Why: the `no-mistakes(review): ...` fix commits stay visible next to the commit they fixed.
A squash would hide what the review found, and a merge commit adds noise without adding information.

## Conventional commits

Commit subjects use `feat:`, `fix:`, `docs:`, `chore:`, `ci:`, `test:`, with an optional scope such as `feat(logs):`.
The body says why, and ends with `Refs #N` or `Closes #N` when an issue exists.

Why: release-please will read these to compute versions and changelogs ([distribution.md](distribution.md)), so the habit needs to be in place before the tool is.

## PR descriptions

A PR body has two parts:

1. A short goal paragraph: what the user or agent can do after this PR that they could not before.
2. Decision bullets: each choice a reviewer might question, with the reason in one line.

Example decision bullet: "`variables list` prints names only. Values are secrets, and `get` exists for the one value an agent needs."

Why: the reviewer, human or bot, judges the PR against its stated intent. A list of changed files does not give it one.

## Gap issues

When an agent finds an operation the axi does not wrap, it falls back to the plain CLI, finishes the task, and files an issue labeled `agent-reported-gap`.
The issue template lives in the shipped skill ([railway-axi skill](https://github.com/simkimsia/railway-axi/blob/main/skills/railway-axi/SKILL.md)): what I tried, what worked instead, what the agent needed from the output, and the task context.
It also asks two questions:

- **Plain CLI result, with the exact argv the axi forwarded:** works, same failure, or n/a.
- **If same failure:** should the axi shape the arguments, map the error, or document the limit?

One issue per missing subcommand. Comment on an existing issue instead of opening a duplicate.
For the enforcement side of this, see [using-others-axis.md](using-others-axis.md).

## Inherited gaps

"The plain CLI fails too" does not close a gap. The agent hit the failure through the axi, so the axi is the place to fix or explain it.
Triage every gap into one of three outcomes:

1. **Axi-caused.** The axi's argv translation produces the failure. Fix the translation.
2. **Inherited.** The vendor CLI really behaves this way. The axi still owes one of: a workaround (shape the argv so the vendor call succeeds), a mapped error that names the cause and the next step, or a documented limit in `--help` and the skill. Label the issue `inherited-gap` and keep it open until one of those ships. Link the vendor's own issue if there is one.
3. **Out of reach.** Nothing the axi sends can avoid it, for example an API limit. Document it in `--help` and the skill, then close the issue with a pointer to that line.

The bar: the agent either succeeds, or gets an error it can act on in one retry (AXI principle: actionable errors).

Prove the classification with the forwarded argv (`AXI_DEBUG=1`, [architecture.md](architecture.md#one-module-spawns-the-vendor-binary)), not with a query retyped by hand.
The worked example is gh-axi [#163](https://github.com/kunchenguid/gh-axi/issues/163), closed as a vendor bug, and [#188](https://github.com/kunchenguid/gh-axi/pull/188), which fixed it in the wrapper.
