# Conditional modules

Some modules are required only when a repo has a certain kind of command.
Each section says when it becomes required and which repo shows it.

## Secret redaction

**Required when** any command can see a secret value: environment variables, API keys, tokens, connection strings.

The rule from [railway-axi/VISION.md](https://github.com/simkimsia/railway-axi/blob/main/VISION.md): a value is printed only by a command that names one variable, and never appears in lists, errors, or logs.

How railway-axi does it:

- `variables list` prints names only. `get` prints the one value asked for. `set` reports names and whether a redeploy started, never the value.
- Vendor calls that may carry secrets pass `SecretOptions` to the spawner ([src/railway.ts](https://github.com/simkimsia/railway-axi/blob/main/src/railway.ts)), which masks them out of any error.
- `secretCandidates` in [src/commands/variables.ts](https://github.com/simkimsia/railway-axi/blob/main/src/commands/variables.ts) is an allowlist: any argv token not known to be safe is treated as a possible secret and masked.
- [test/variables-leak.test.ts](https://github.com/simkimsia/railway-axi/blob/main/test/variables-leak.test.ts) fakes the binary and sweeps every flag and failure mode to prove no value leaks.

Why an allowlist: a denylist misses the next flag someone adds. With an allowlist, a new flag is masked until someone proves it safe.

## Ambiguity refusal

**Required when** a command targets one resource inside a group and the target can be inferred: one service in an environment, one site in a team, one Worker in an account.

The rule: when several could match and none is named by flag or linked to the current directory, refuse and list the names. Never guess.

How railway-axi does it: [`assertServiceUnambiguous` in src/scope.ts](https://github.com/simkimsia/railway-axi/blob/main/src/scope.ts) uses the cwd-linked service as the tiebreak, and otherwise throws `VALIDATION_ERROR` with each candidate name in the help list.

Why: a guessed target on a read wastes a call. A guessed target on a write changes the wrong thing.

## Write-command safety

**Required when** the axi adds any command that changes vendor state.

The rules, from the Safety sections of [railway-axi](https://github.com/simkimsia/railway-axi/blob/main/VISION.md) and [cloudflare-axi](https://github.com/simkimsia/cloudflare-axi/blob/main/VISION.md):

- Read commands are the default and never change state.
- Writes are explicit verbs (`set`, `create`, `deploy`) and print what changed, including side effects such as a redeploy.
- A command that deletes or overwrites requires the target named in full. It never infers it from context.
- Mutations should be idempotent where the vendor allows it (AXI §6).
- Validate locally before the call when the vendor would accept something harmful. cloudflare-axi's `assertDeployableDir` in [src/commands/pages.ts](https://github.com/simkimsia/cloudflare-axi/blob/main/src/commands/pages.ts) refuses an empty directory because `wrangler pages deploy` would publish it without complaint.
- Each write needs a documented live smoke procedure ([testing.md](testing.md)).

Destructive commands stay unwrapped until there is a clear need.
The agent skill lists them under "Deliberately not wrapped (do not file)" so agents use the plain CLI and do not open issues for them ([railway-axi skill](https://github.com/simkimsia/railway-axi/blob/main/skills/railway-axi/SKILL.md)).

## Session hooks (AXI §7)

**Required when** the tool has directory-scoped state worth showing at session start, and the dashboard fits in a small token budget.

AXI §7 makes a session hook the primary integration and the skill secondary.
gh-axi implements it as an explicit `gh-axi setup hooks` command using the SDK's `installSessionStartHooks` ([src/commands/setup.ts](https://github.com/kunchenguid/gh-axi/blob/main/src/commands/setup.ts)).

None of my four repos have it yet.
The candidates are railway-axi and netlify-axi, where the linked project or site is useful context.
When adding it, follow gh-axi: opt-in setup command only, idempotent, path repair, Claude Code, Codex, and OpenCode by default.
