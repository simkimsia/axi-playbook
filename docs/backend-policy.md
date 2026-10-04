# Backend policy

The backend is what the axi calls to get work done: the vendor CLI, the vendor's HTTP API, or both.
Each repo states its policy in the first lines of `VISION.md`, so a triage agent can check a PR against it.

## CLI first

The default backend is the official vendor CLI.
A command reaches for anything else only when the CLI cannot serve it.

Why:

- The CLI owns login, token refresh, config files, and the directory link, and the axi inherits all of it.
- Everything the axi does, the user can redo with the plain CLI. That is what makes the fallback in the agent skill possible.
- Vendor CLI releases track vendor API changes, so the axi has less to keep current.

## When REST or GraphQL is allowed

An API call is allowed when one of these holds:

- The CLI has no command for the surface at all.
- One API query replaces several CLI calls per row (an N+1 the agent would otherwise pay for).

In both cases the call must hit a documented public endpoint and reuse the credentials the CLI already holds.

Examples:

- [cloudflare-axi/VISION.md](https://github.com/simkimsia/cloudflare-axi/blob/main/VISION.md) puts DNS and Email Routing in scope through the REST API because `wrangler` has no surface for them. All REST calls go through one module, [src/api.ts](https://github.com/simkimsia/cloudflare-axi/blob/main/src/api.ts), the same "one door" rule as the spawner.
- [railway-axi/VISION.md](https://github.com/simkimsia/railway-axi/blob/main/VISION.md) allows the public GraphQL API through `railway api`, which keeps even API calls inside the CLI's auth. No command uses it yet.
- netlify-axi's `whoami` uses `netlify api getCurrentUser` ([src/commands/whoami.ts](https://github.com/simkimsia/netlify-axi/blob/main/src/commands/whoami.ts)), the CLI's own API passthrough, because `netlify status` exits 1 when the directory is unlinked.

## Auth reuse

An axi never asks for its own token and never stores one.

- railway-axi uses the session `railway login` holds.
- cloudflare-axi reads `CLOUDFLARE_API_TOKEN` if set, else wrangler's own OAuth token from its config file, and runs `wrangler whoami` to refresh it when expired ([src/credentials.ts](https://github.com/simkimsia/cloudflare-axi/blob/main/src/credentials.ts)).
- calcom-axi passes the environment through so `CAL_API_KEY` and `calcom login` both work.

Why: a second credential store is a second thing to leak, expire, and explain.
When a scoped token is needed for an API the CLI login does not cover (cloudflare-axi's DNS records), the error says which env var to set.

## MCP stance

railway-axi does not call Railway's MCP server, and the rule is written into its VISION.md.

Why:

- Every MCP tool maps to an existing CLI command or public API operation, so MCP adds no capability.
- The benchmark compares the axi against the raw CLI and against the vendor MCP server as separate arms. If the axi called MCP, the arms would overlap and the comparison would mean less.

I apply the same stance to new repos unless a vendor exposes something only through MCP.

## Counter-model: API-only

[supabase-axi](https://github.com/laizhenyoong/supabase-axi) by laizhenyoong wraps the Supabase Management API directly, with no CLI underneath.
That is a valid design, and the right one when a vendor's CLI is mostly about local development, as Supabase's is.

The trade-off is that the axi then owns auth, pagination, and API drift itself, and the plain-CLI fallback is weaker.
If a vendor pushes me toward API-only, I write that down in VISION.md as an explicit exception.

## Gaps

netlify-axi and calcom-axi have no VISION.md, so their backend stance is unwritten.
Both follow CLI first in practice. Writing it down is on the drift list in [conformance.md](conformance.md).
