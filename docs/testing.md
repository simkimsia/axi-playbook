# Testing

## Tests are offline

Vitest tests in `test/` never spawn the real vendor binary and never touch the network.
They feed captured vendor output to the exported parse and mapping helpers.

Why: tests must pass in CI with no vendor login, and a live account changes under you.
A test that depends on the account's current projects fails for reasons unrelated to the code.

When a test needs process behavior (exit codes, stderr, argv), fake the binary.
railway-axi's [test/variables-leak.test.ts](https://github.com/simkimsia/railway-axi/blob/main/test/variables-leak.test.ts) does this to prove secrets never leak.

## Fixtures are verbatim

Every error pattern is backed by a verbatim stderr sample in `test/errors.test.ts`, with a comment saying which command and CLI version produced it.
See [calcom-axi/test/errors.test.ts](https://github.com/simkimsia/calcom-axi/blob/main/test/errors.test.ts).

Why: a regex written against what you think stderr says will miss the real text.
Capture the real output first, then write the pattern, then the test.

JSON fixtures mirror the real `--json` shape, scrubbed of account data.
Keep the awkward parts the vendor really emits: ANSI codes, banners on stdout, a literal `undefined`, display-oriented keys.

## Record CLI quirks in AGENTS.md

When a run reveals a quirk, add it to the `<Vendor> CLI notes` section of `AGENTS.md` with the CLI version checked.
Examples worth copying:

- `wrangler whoami` exits 0 when logged out ([cloudflare-axi/AGENTS.md](https://github.com/simkimsia/cloudflare-axi/blob/main/AGENTS.md)).
- An unauthenticated `netlify` call starts a browser login and hangs. Probe auth with an invalid `NETLIFY_AUTH_TOKEN` instead ([netlify-axi/AGENTS.md](https://github.com/simkimsia/netlify-axi/blob/main/AGENTS.md)).

## Live smoke for writes

Offline tests cannot prove a write works against the vendor.
Every write command gets a documented smoke procedure in `AGENTS.md`, run by hand after touching that command.

The pattern, from cloudflare-axi's "Live smoke procedure for Pages writes":

1. Create a throwaway resource with an obvious name (`cloudflare-axi-smoke`).
2. Run the write through the axi.
3. Read it back through the axi.
4. Delete the throwaway resource with the plain CLI.

Why plain CLI for cleanup: delete is deliberately not wrapped, and cleanup should not depend on the code under test.

railway-axi's `variables set` has no written smoke procedure yet; see [conformance.md](conformance.md).

## Before pushing

```sh
pnpm run build
pnpm run format:check
pnpm test
```

CI runs the same three steps ([repo-skeleton.md](repo-skeleton.md#ci)).
