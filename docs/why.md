# Why build an axi

## The problem

Coding agents operate cloud services through shell commands.
Vendor CLIs are written for people at a terminal, so agents pay for that in several ways:

- Output is wide tables, colored text, or JSON objects with 100 keys per row.
- Errors are prose on stderr, sometimes with ANSI codes, with no stable code to branch on.
- Some commands prompt, open a browser, or stream forever, and the agent hangs.
- Unknown flags are sometimes ignored, so the agent believes a filter applied when it did not.

Each of these costs tokens, extra calls, or a wrong answer.
An axi is a thin wrapper that fixes them without changing what the vendor can do.

## Why a CLI and not an MCP server

Agents are already good at shell.
A CLI works in every agent harness, needs no server process, composes with pipes, and can be installed with one command.

A CLI also lets the wrapper reuse the vendor's own login.
The agent never handles a token, and the user logs in once with the tool they already trust.

The AXI benchmarks upstream ([bench-github](https://github.com/kunchenguid/axi/tree/main/bench-github), [bench-browser](https://github.com/kunchenguid/axi/tree/main/bench-browser)) compare these interface styles directly.
My own per-vendor benchmarks follow [axi-bench](https://github.com/simkimsia/axi-bench); see [ecosystem.md](ecosystem.md).

## Why wrap the vendor CLI instead of calling the API

The vendor CLI already handles auth, token refresh, config files, and the directory-linked project.
Wrapping it means the axi inherits all of that and only has to fix the interface.

It also keeps the wrapper honest.
If the axi can do something, the vendor CLI can do it too, so a user can always fall back.
The exceptions, and the rules for them, are in [backend-policy.md](backend-policy.md).

## gh-axi as the reference

[gh-axi](https://github.com/kunchenguid/gh-axi) is the reference implementation, written by the author of the spec.
When I add a capability, I check how gh-axi solved the analogous problem first.

Concrete things copied from it:

- The ordered error pattern table ([`mapGhError` in src/errors.ts](https://github.com/kunchenguid/gh-axi/blob/main/src/errors.ts)).
- The `formatError` hook in `src/cli.ts`.
- The skill that defers to `--help` instead of listing commands ([skills/gh-axi/SKILL.md](https://github.com/kunchenguid/gh-axi/blob/main/skills/gh-axi/SKILL.md)).
- The release setup I plan to adopt ([distribution.md](distribution.md)).

Every vendor repo's `AGENTS.md` says this in its first section, so agents working there pick it up too.

## Scope of an axi

An axi exposes what the vendor already provides, more ergonomically.
It does not add features the vendor lacks, and it does not embed workflow logic that belongs in the calling agent.

That line is written into each repo's `VISION.md` ([railway-axi](https://github.com/simkimsia/railway-axi/blob/main/VISION.md), [cloudflare-axi](https://github.com/simkimsia/cloudflare-axi/blob/main/VISION.md)) so a triage agent can rule on proposals against it.
