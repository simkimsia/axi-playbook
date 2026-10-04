# Architecture

Every axi has the same internal shape.
The pieces below are small, and each one exists because of a specific failure it prevents.

## Fast path for --version

`bin/<vendor>-axi.ts` answers bare `-v`, `-V`, and `--version` through `axi-sdk-js/fast-path` before it imports anything else.
Only on a miss does it dynamically import `src/cli.ts`.

Why: agents call `--version` to check a tool is installed, and loading the whole command graph for that wastes time on every session (AXI §10).
`src/version.ts` must stay a leaf module that imports only node builtins, or the fast path silently stops being fast.

Example: [railway-axi/bin/railway-axi.ts](https://github.com/simkimsia/railway-axi/blob/main/bin/railway-axi.ts).

## One module spawns the vendor binary

Exactly one file runs the vendor CLI: [src/railway.ts](https://github.com/simkimsia/railway-axi/blob/main/src/railway.ts), [src/wrangler.ts](https://github.com/simkimsia/cloudflare-axi/blob/main/src/wrangler.ts), [src/netlify.ts](https://github.com/simkimsia/netlify-axi/blob/main/src/netlify.ts), [src/calcom.ts](https://github.com/simkimsia/calcom-axi/blob/main/src/calcom.ts).
Commands call its helpers (`railwayJson`, `wranglerExec`, and so on) and never spawn directly.

Why: error mapping, JSON parsing quirks, environment handling, and secret redaction all live in one place.
A missing binary (`ENOENT`) maps to `<VENDOR>_NOT_INSTALLED` with an install hint, so the agent knows to ask the user instead of retrying.

## Errors: ordered patterns, first match wins

`src/errors.ts` defines `AxiError(message, code, suggestions[])` and a `map<Vendor>Error(stderr, exitCode)` function.
The mapper walks a list of regex patterns in order and returns on the first hit.
Unmatched stderr becomes `UNKNOWN` with the first stderr line as the message.

Why: order is the contract.
A narrow pattern ("no service linked") must sit above a broad one ("not found"), and a comment next to each pattern says which real stderr it matches.
This is the same rule as gh-axi's [`mapGhError`](https://github.com/kunchenguid/gh-axi/blob/main/src/errors.ts).

Only `VALIDATION_ERROR` exits 2. Everything else exits 1.
Exit 2 tells the agent it called the tool wrong and should fix the call, not retry.

Examples: [railway-axi/src/errors.ts](https://github.com/simkimsia/railway-axi/blob/main/src/errors.ts), and [cloudflare-axi/src/errors.ts](https://github.com/simkimsia/cloudflare-axi/blob/main/src/errors.ts), which strips ANSI first and also maps REST API error envelopes into the same codes.

## Error code vocabulary

Agents learn codes across tools, so the same situation should get the same code in every axi.
Proposed canonical set:

| Code | Meaning | Exit |
| --- | --- | --- |
| `AUTH` | not logged in, or credentials rejected | 1 |
| `NOT_LINKED` | the command needs a directory-linked project, site, or Worker and there is none | 1 |
| `NOT_FOUND` | a named resource does not exist | 1 |
| `ALREADY_EXISTS` | a create hit a taken name | 1 |
| `RATE_LIMITED` | the vendor throttled the call | 1 |
| `CONFIG` | a local config file is broken | 1 |
| `VALIDATION_ERROR` | bad flags or arguments, rejected by the axi or the vendor CLI | 2 |
| `<VENDOR>_NOT_INSTALLED` | the vendor binary is not on PATH | 1 |
| `UNKNOWN` | nothing matched; message is the first stderr line | 1 |

Current drift:

- cloudflare-axi uses `NOT_CONFIGURED` where the others use `NOT_LINKED` (wrangler needs a Worker name from a config in cwd). Its AGENTS.md calls it "the analog of railway-axi's `NOT_LINKED`". Proposal: rename to `NOT_LINKED`.
- gh-axi uses `AUTH_REQUIRED`, `FORBIDDEN`, and `REPO_NOT_FOUND`. My four use `AUTH` and `NOT_FOUND`. I am keeping `AUTH` because all four already agree on it.
- `CONFIG` exists only in netlify-axi and `RATE_LIMITED` only in calcom-axi. Both are fine; other repos add them when the vendor has the failure.

## Unknown input is rejected by name

[src/args.ts](https://github.com/simkimsia/railway-axi/blob/main/src/args.ts) has `take*` helpers that pull the flags a command knows.
Whatever is left is rejected by `assertNoArgs` with `VALIDATION_ERROR`, naming each leftover token, before any vendor call.

Why: a silently ignored `--status failed` returns unfiltered data that the agent will trust (AXI §6).
cloudflare-axi names its leftover check `rejectExtraArgs` ([src/args.ts](https://github.com/simkimsia/cloudflare-axi/blob/main/src/args.ts)) and adds one rule worth keeping everywhere: take value flags before positionals, so a flag's value is never mistaken for a positional.

## TOON output, truncation, and --full

Commands return TOON strings built with the helpers in `src/toon.ts`, identical in railway, cloudflare, and netlify.
TOON cuts tokens compared with JSON (AXI §1), and the helpers keep list schemas to a few fields (AXI §2).

Long values are a separate problem (AXI §3).
calcom-axi's [`Truncator` in src/toon.ts](https://github.com/simkimsia/calcom-axi/blob/main/src/toon.ts) is the pattern to adopt: every shortened value carries its original size, and a renderer emits one `--full` hint only when it actually cut something.
The other three repos do not truncate yet.

## formatError hook

`src/cli.ts` passes a `formatError` hook to `runAxiCli`.
It wraps any non-AxiError as `UNKNOWN`, encodes `{error, code, help}` as TOON on stdout, and sets the exit code.

Why: the SDK's default formatter only recognizes the SDK's own `AxiError` class, so without the hook a vendor error loses its code and suggestions.
gh-axi does the same. See [railway-axi/src/cli.ts](https://github.com/simkimsia/railway-axi/blob/main/src/cli.ts).

## Help, home, and next-step hints

- No arguments runs `src/commands/home.ts`, a short dashboard of the linked project or recent items (AXI §8).
- `--help` prints a compact command index; `<command> --help` prints that command's usage. The SDK routes `<command> <sub> --help` to the top-level command's help, so one text per command is enough (AXI §10).
- Every output and error ends with a `help` list of concrete commands to run next, using real names from the output where possible (AXI §9).
- Empty results say so with a count of 0 and a next step, never blank output (AXI §5).
- `update` is a reserved SDK built-in, so self-update needs no code.
