# Project agent memory

Project-intrinsic knowledge for agents working on <vendor>-axi.

## What this is

An AXI-compliant wrapper around the <Vendor> CLI (`<cli>`), built on
`axi-sdk-js` (`runAxiCli` in `src/cli.ts`) and deliberately modeled on the
reference implementation [gh-axi](https://github.com/kunchenguid/gh-axi) and
the sibling [railway-axi](https://github.com/simkimsia/railway-axi). When
adding a capability, check how gh-axi solved the analogous problem first, and
follow the AXI principles (the `axi` skill in the upstream `kunchenguid/axi`
repo).

## Architecture

- `bin/<vendor>-axi.ts`: entrypoint; answers bare `-v`/`-V`/`--version` via
  `axi-sdk-js/fast-path` before dynamically importing `src/cli.ts`.
  `src/version.ts` must stay a LEAF module (node builtins only) or the fast
  path silently stops being fast.
- `src/<cli>.ts`: sole place that spawns the `<cli>` binary. Non-zero exits
  route through `map<Vendor>Error`; a missing binary maps to
  `<VENDOR>_NOT_INSTALLED`.
- `src/errors.ts`: `map<Vendor>Error` walks `patterns` in order and returns on
  the first regex hit, so order is the contract: narrow patterns before broad
  ones (same rule as gh-axi's `mapGhError`). Verify new patterns against real
  `<cli>` stderr before adding them.
- `src/args.ts`: commands pull the flags they know with the `take*` helpers,
  then `assertNoArgs` rejects whatever is left by name with exit code 2
  before any `<cli>` call (AXI §6). A flag is never accepted silently.
- Commands live in `src/commands/`, return TOON strings via `src/toon.ts`
  helpers; errors render through the `formatError` hook in `src/cli.ts`
  because the SDK's default formatter only recognizes its own AxiError class.

## <Vendor> CLI notes (verified against <cli> <version>)

- <Output shape of each wrapped command: JSON, NDJSON, or text only.>
- <Which commands are directory-scoped (linked project) and which are account-scoped.>
- <Exit codes that lie, commands that prompt or hang, banners on stdout.>
- <Verbatim stderr for auth failure, not found, and not linked.>
- The SDK ships `update` as a reserved built-in, so `<vendor>-axi update`
  works with no code here; the npm package name resolves from `package.json`.

## Live smoke procedure for writes

<Delete this section until the first write command exists.>
Tests are offline, so after touching a write command, smoke it for real on a
throwaway resource named `<vendor>-axi-smoke`: create, write, read back with
`<vendor>-axi`, then delete with plain `<cli>`.

## Conventions

- pnpm, Node >= 20, ES modules, TypeScript Node16 resolution
  (import specifiers end in `.js`), Vitest tests in `test/`.
- Tests are OFFLINE: they feed captured real `<cli>` output as fixtures and
  never spawn the real binary or touch the network.
- Conventional commit messages (`feat:`, `fix:`, `docs:`) for release-please.

## Maintaining this file

Keep entries concise and durable; point at the authoritative file rather than
restating what the code shows. Prefer rewriting or pruning over appending.
