# Conformance

Which repo follows which convention, checked against each repo's `main` on 2026-10-04.

Legend: **yes** follows the convention, **no** missing, **diff** present but different from the convention, **n/a** not required yet (see [conditional-modules.md](conditional-modules.md)), **PR** rolling out via an open PR, not merged yet.

## Scaffold and architecture

| Convention | railway | cloudflare | netlify | calcom |
| --- | --- | --- | --- | --- |
| Shared scaffold (bin, tsconfig, package.json) | yes | yes | yes | yes |
| `--version` fast path, leaf `src/version.ts` | yes | yes | yes | yes |
| Single spawner module | yes | yes | yes | yes |
| `<VENDOR>_NOT_INSTALLED` | yes | yes | yes | yes |
| Ordered error patterns, first match wins | yes | yes | yes | yes |
| "No linked context" code is `NOT_LINKED` | yes | diff (`NOT_CONFIGURED`) | yes | n/a |
| Only `VALIDATION_ERROR` exits 2 | yes | yes | yes | yes |
| `formatError` hook | yes | yes | yes | yes |
| Unknown args rejected by name | yes | diff (`rejectExtraArgs`) | yes | yes |
| Home dashboard | yes | yes | yes | yes |
| Truncation with `--full` (AXI §3) | no | no | no | yes |
| Session hooks (AXI §7) | no | no | no | no |

## Backend and safety

| Convention | railway | cloudflare | netlify | calcom |
| --- | --- | --- | --- | --- |
| Backend stance written in VISION.md | yes | yes | no | no |
| REST calls through one module | n/a | yes (`src/api.ts`) | n/a | n/a |
| Secret redaction | yes | n/a | n/a | n/a |
| Ambiguity refusal | yes | n/a | n/a | n/a |
| Write commands follow Safety rules | yes | yes | n/a | n/a |
| Live smoke procedure for writes in AGENTS.md | no | yes | n/a | n/a |

## Repo files and process

| Convention | railway | cloudflare | netlify | calcom |
| --- | --- | --- | --- | --- |
| README sections (Status to License) | yes | yes | yes | yes |
| AGENTS.md sections, CLAUDE.md = `@AGENTS.md` | yes | yes | yes | yes |
| Shipped skill, `user-invocable: false` | yes | yes | yes | yes |
| Skill generated from CLI text | no | no | no | no |
| VISION.md | yes | yes | no | no |
| CI workflow on Node 24 ([template](../templates/.github/workflows/ci.yml)) | PR | PR | PR | PR |
| Triage crewmate | no | yes | no | no |
| `.prettierignore` | PR | PR | PR | PR |
| README install shape ([template](../templates/README.md#install)) | PR | PR | PR | PR |
| PRs through no-mistakes | yes | no | n/a (no PRs) | n/a (no PRs) |
| Scoped npm name | yes | yes | yes | yes |
| Published on npm (0.1.0) | yes | yes | yes | yes |
| Release automation | no | no | no | no |
| `bench/` | no | no | no | no |
| Listed in upstream catalog | yes | yes | yes | yes |

## Known drift to fix

1. **README install section.** All four move to the shape in the [README template](../templates/README.md#install): `pnpm add -g @simkimsia/<name>` first, `npx` as the no-install option, and a clone path that uses `pnpm -C` and `pnpm add -g link:` because pnpm ignores `--prefix` ([cloudflare-axi#12](https://github.com/simkimsia/cloudflare-axi/issues/12)). Rolling out via PRs.
2. **Error code for no linked context.** cloudflare-axi's `NOT_CONFIGURED` should become `NOT_LINKED` ([architecture.md](architecture.md#error-code-vocabulary)).
3. **Backend stance.** netlify-axi and calcom-axi need a VISION.md that states CLI first and when API calls are allowed.
4. **CI.** All four get the same `.github/workflows/ci.yml` from [templates](../templates/.github/workflows/ci.yml), byte for byte, on Node 24. Rolling out via PRs.
5. **`.prettierignore`.** All four get the one-line file, `pnpm-lock.yaml`, alongside CI. pnpm can write a lockfile Prettier would reformat, and nobody should hand-format a lockfile. Rolling out via PRs.
6. **Live smoke for `variables set`.** railway-axi's only write has no written smoke procedure.
7. **Truncation.** Port calcom-axi's `Truncator` to the other three when a command can return long text (railway-axi `logs` is the first candidate).
8. **Args helper name.** Pick one of `assertNoArgs` and `rejectExtraArgs` for the "reject leftovers" step and use it in all four.

Done since the first audit: the scoped npm name is merged in all four ([railway-axi#13](https://github.com/simkimsia/railway-axi/pull/13), [cloudflare-axi#13](https://github.com/simkimsia/cloudflare-axi/pull/13), [netlify-axi#2](https://github.com/simkimsia/netlify-axi/pull/2), [calcom-axi#2](https://github.com/simkimsia/calcom-axi/pull/2)), and all four are published on npm as `@simkimsia/<name>@0.1.0` ([distribution.md](distribution.md)).

Larger items, not drift: session hooks, a generated skill, release-please, and `bench/` in each repo.
