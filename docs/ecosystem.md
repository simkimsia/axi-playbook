# Ecosystem

An axi is more than the binary.
These pieces make it discoverable, measurable, and maintainable.

## Shipped agent skill

Each repo ships `skills/<vendor>-axi/SKILL.md` inside the npm package, installable with:

```sh
npx skills add simkimsia/<vendor>-axi --skill <vendor>-axi -g
```

The skill is a discovery stub. It tells the agent to prefer the axi over the plain CLI and then defers to `<vendor>-axi --help` for current commands, because an installed copy of a command list goes stale.
The shape follows [gh-axi's skill](https://github.com/kunchenguid/gh-axi/blob/main/skills/gh-axi/SKILL.md), with sections my repos add:

- **Setup**: install from a clone, and what `<VENDOR>_NOT_INSTALLED` or `NOT_LINKED` means.
- **When the axi cannot do it**: fall back to the plain CLI, finish the task, then search for and file a gap issue with a fixed template.
- **Deliberately not wrapped (do not file)**: destructive commands excluded by design, so agents do not open issues for them.

The frontmatter sets `user-invocable: false`, because the skill is for agents to load on intent, not a slash command.
Template: [templates/skills/vendor-axi/SKILL.md](../templates/skills/vendor-axi/SKILL.md). Live example: [railway-axi](https://github.com/simkimsia/railway-axi/blob/main/skills/railway-axi/SKILL.md).

AXI §7 recommends generating the skill from the same text the home view prints, with a CI check for drift. gh-axi does this with a `build:skill` script. My four are hand-written today; see [conformance.md](conformance.md).

## Catalog entry

Every axi gets one entry in the community catalog, [catalog.yaml](https://github.com/kunchenguid/axi/blob/main/catalog.yaml) in kunchenguid/axi.
The steps are in upstream [CONTRIBUTING.md](https://github.com/kunchenguid/axi/blob/main/CONTRIBUTING.md): add the entry, run `pnpm run docs:gen`, commit all three files, and push through no-mistakes from a fork.

How I write the entry:

- `description` says what an agent can do with it today, then "Wraps the `<Vendor>` CLI". It describes shipped commands only, so a reviewer can verify every claim against the source.
- Upstream [VISION.md](https://github.com/kunchenguid/axi/blob/main/VISION.md) requires an independent source review before admission. That review comes from the reviewer side. I do not write an admission review into my own PR.

All four are listed: kunchenguid/axi PRs [#173](https://github.com/kunchenguid/axi/pull/173) (railway), [#174](https://github.com/kunchenguid/axi/pull/174) (netlify), [#176](https://github.com/kunchenguid/axi/pull/176) (cloudflare), [#190](https://github.com/kunchenguid/axi/pull/190) (calcom).

## Scoring

Before a catalog PR and after large changes, I score the axi on a fixed set of general axes drawn from the AXI principles, plus a per-domain worksheet of the operations agents need from that vendor.
For each axis: find the source line that implements it, run the command that shows it, and record pass, partial, or missing with the evidence.
I also score the same axes on gh-axi as a baseline, so "partial" is judged against the reference and not against an ideal.

Gaps found this way become issues on the repo, not notes in a scorecard.

## Benchmarks

[axi-bench](https://github.com/simkimsia/axi-bench) is my benchmark framework on the Harbor task format.
Each vendor repo gets a `bench/` directory copied from [templates/vendor-bench](https://github.com/simkimsia/axi-bench/tree/main/templates/vendor-bench): tasks, conditions (one per tool surface), a pinned environment, and live fixtures.

The arms compare the axi against the raw vendor CLI and, where one exists, the vendor's MCP server.
That is why railway-axi refuses to call MCP itself ([backend-policy.md](backend-policy.md#mcp-stance)).
Read axi-bench [docs/methodology.md](https://github.com/simkimsia/axi-bench/blob/main/docs/methodology.md) before publishing a result.

No vendor repo has a `bench/` yet.

## VISION.md and the triage crewmate

`VISION.md` has three `##` sections: Scope, Interface, Safety. One rule per line.
Each heading is a rule a triage agent rules on (aligns, does not align, cannot tell) for every issue and PR, with evidence.
That is why the file is short and every line is checkable: a vague line produces "cannot tell" forever.

Examples: [railway-axi/VISION.md](https://github.com/simkimsia/railway-axi/blob/main/VISION.md), [cloudflare-axi/VISION.md](https://github.com/simkimsia/cloudflare-axi/blob/main/VISION.md). Template: [templates/VISION.md](../templates/VISION.md).

cloudflare-axi runs the crewmate as [.github/workflows/repo-triage.yml](https://github.com/simkimsia/cloudflare-axi/blob/main/.github/workflows/repo-triage.yml):

- Cron every 4 hours, always dry-run: verdicts go to the run summary and an artifact.
- Manual dispatch with `mode=live` posts the verdict comment and a `ready-for-pr` label.
- It never merges, closes, or opens PRs.
- Every verdict carries a provenance stamp (model, harness, run id), so a wrong verdict can be traced.

The other three repos have no crewmate yet.
