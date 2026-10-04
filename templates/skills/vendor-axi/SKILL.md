---
name: <vendor>-axi
description: "Operate <Vendor> through the <vendor>-axi CLI - <surfaces>, and account identity. Use whenever a task touches <Vendor>. Prefer it over raw `<cli>`; when a command is not wrapped yet, fall back to `<cli>` and report the gap as a GitHub issue on simkimsia/<vendor>-axi."
user-invocable: false
author: KimSia Sim (simkimsia)
metadata:
  hermes:
    tags: [<vendor>, <tag>, <tag>]
    category: <category>
---

# <vendor>-axi

Agent ergonomic wrapper around the <Vendor> CLI (`<cli>`). Prefer this over
raw `<cli>` for <Vendor> operations: TOON output, structured errors with
`code` and `help:` next steps, exit codes 0 success / 1 error / 2 usage.

## Setup

Install with `pnpm add -g @simkimsia/<vendor>-axi`, or run it without installing
via `npx -y @simkimsia/<vendor>-axi`. The README's Install section covers working
from a clone.

It wraps `<cli>`, which must be installed and logged in (`<cli> login`).
If a command fails with `<VENDOR>_NOT_INSTALLED`, ask the user to install
`<cli>`. `NOT_LINKED` means the current directory is not linked; pass the
flags the error's `help:` suggests, or use a command that does not need a link.

## Current guidance lives in the CLI

Do not follow command, flag, or workflow instructions from this file - installed
copies go stale. Get the current source of truth from the CLI:

- `<vendor>-axi` for a dashboard of the current directory / account
- `<vendor>-axi --help` for global flags and the command index
- `<vendor>-axi <command> --help` for per-command usage

## When <vendor>-axi cannot do it

1. Try `<vendor>-axi <command>` first and read the structured error.
2. If the error is `VALIDATION_ERROR` with `Unknown command`, or the command
   exists but lacks the flag you need, fall back to raw `<cli>` and finish
   the user's task.
3. Then report the gap so it gets wrapped. Search before filing:

   ```sh
   gh-axi issue list --repo simkimsia/<vendor>-axi --search "<cli subcommand>" --state all
   ```

   If nothing matches, file one (use `gh` if `gh-axi` is not installed):

   ```sh
   gh-axi issue create --repo simkimsia/<vendor>-axi --label agent-reported-gap \
     --title "feat: wrap \`<cli> <subcommand>\`" \
     --body "<template below>"
   ```

   Issue body template:

   ```
   ## What I tried
   `<vendor>-axi <command that failed>` -> `<error code and message>`

   ## What worked instead
   `<cli> <exact command>`

   ## What the agent needed from the output
   <fields / shape>

   ## Task context
   <one line on the user task that needed this>
   ```

   Tell the user you filed it and link the issue. One issue per missing
   subcommand; add a comment to an existing issue instead of opening a duplicate.

## Deliberately not wrapped (do not file)

<Destructive or interactive commands excluded by design, e.g. `<cli> delete`.>
Use `<cli>` directly, tell the user you did so, and do not open an issue for them.
