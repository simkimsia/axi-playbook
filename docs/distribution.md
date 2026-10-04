# Distribution

All four repos are published on npm under my scope, as `@simkimsia/<vendor>-axi`.
Version 0.1.0 of each went out on 2026-10-04.
This doc covers the naming rule, the install paths, the publish gotchas I hit, and release automation.

## Check the npm name before naming the repo

The SDK's built-in `update` command resolves the package by its npm name from `package.json`.
If someone else owns that name on npm, `<vendor>-axi update` installs their package.

This has already happened.
The npm name `cloudflare-axi` belongs to a different author (radityasurya), so `update` in an unscoped cloudflare-axi would resolve to their package.

Before creating a new repo:

```sh
npm view <vendor>-axi name    # E404 means the name is free
```

The convention I settled on: publish every axi under my npm scope, `@simkimsia/<vendor>-axi`, and keep the command name unscoped through `bin` (`"bin": { "<vendor>-axi": ... }`).
A scope I own cannot be taken by someone else, and using it for all four keeps them consistent even where the unscoped name is free.

The rename is done in all four repos: [railway-axi#13](https://github.com/simkimsia/railway-axi/pull/13), [cloudflare-axi#13](https://github.com/simkimsia/cloudflare-axi/pull/13), [netlify-axi#2](https://github.com/simkimsia/netlify-axi/pull/2), and [calcom-axi#2](https://github.com/simkimsia/calcom-axi/pull/2).

If a repo ever has to keep an unscoped name, the SDK's `packageName` option on `runAxiCli` overrides the name `update` resolves.
Decide before the first release, because changing the name later breaks every existing install's `update`.

## Install

The README and the skill give the npm install first:

```sh
pnpm add -g @simkimsia/<vendor>-axi
```

`npx -y @simkimsia/<vendor>-axi --help` runs it without installing.
`<vendor>-axi update` updates a global install.

To work on an axi from a clone:

```sh
git clone https://github.com/simkimsia/<vendor>-axi
pnpm -C <vendor>-axi install
pnpm -C <vendor>-axi run build
pnpm add -g link:$PWD/<vendor>-axi   # puts `<vendor>-axi` on PATH
```

Use `-C` (pnpm's `--dir`), not `--prefix`.
pnpm ignores `--prefix` for `install` and `link`, so `pnpm --prefix <vendor>-axi link --global` links the current directory instead and the axi never lands on PATH ([cloudflare-axi#12](https://github.com/simkimsia/cloudflare-axi/issues/12)).
`pnpm add -g link:` takes an explicit path, so it links the right directory.

## Publishing by hand

Until release automation exists, I publish from a terminal.
These are the gotchas from the first publish on 2026-10-04.

Pass the package directory as an argument:

```sh
pnpm publish <dir> --no-git-checks --otp=<code>
```

pnpm ignores `--prefix` here too.
Its git-branch check reads the current directory, not the package directory, so publishing from outside the repo needs `--no-git-checks`.

npm needs `--otp` when publishing from a non-interactive shell, such as an agent's shell.
It cannot prompt for the one-time code, so pass it on the command line.

A new scoped package can return 404 for about two minutes after publishing.
Wait before concluding the publish failed or retrying it.

npm puts the account email into the package metadata, where anyone can read it.
Use a forwarding alias as the npm account email, not a personal address.

## Releases: release-please and trusted publishing

Status: rolling out via PRs, starting with railway-axi. Nothing is merged yet, so every repo still publishes by hand (above).
The model is gh-axi, and the files are in [templates](../templates):

- [release-please-config.json](../templates/release-please-config.json) sets the package name, `@simkimsia/<vendor>-axi`, and keeps pre-1.0 bumps small (`feat` bumps the patch, a breaking change bumps the minor).
- [.release-please-manifest.json](../templates/.release-please-manifest.json) is seeded with the version already on npm, `0.1.0`.
- [release-please.yml](../templates/.github/workflows/release-please.yml) runs on every push to `main`.
- [guard-generated-files.yml](../templates/.github/workflows/guard-generated-files.yml) fails any human PR that edits `CHANGELOG.md` or `.release-please-manifest.json`, because release-please owns them.
  It counts only modified or deleted files (`--diff-filter=MD`), which is one change from gh-axi's copy.
  The PR that sets up release-please has to create the manifest, and gh-axi's version fails that PR; gh-axi never hit it because its guard came after its manifest.
  Once the files exist, every hand edit is still caught.
- [ci.yml](../templates/.github/workflows/ci.yml) gets a `paths-ignore` block for those same files.

How a release happens:

1. Conventional commits land on `main` ([process.md](process.md#conventional-commits)).
2. release-please opens or updates one release PR that bumps `package.json`, the manifest, and `CHANGELOG.md` from those commits.
3. Merging that PR tags the release, and the same workflow builds and runs `npm publish --access public --provenance`.

Publishing uses npm trusted publishing (OIDC).
There is no `NPM_TOKEN` secret: the workflow has `id-token: write`, and npm trusts that one workflow in that one repo.
Trusted publishing needs npm 11.5.1 or later, and the npm bundled with the runner's Node 24 can be older, so the workflow runs `npm install -g npm@latest` before publishing.
Provenance ties each npm version to the commit and workflow run that built it.

One-time setup per package, done by hand on npmjs.com before the first automated release: open the package's Settings, add a Trusted Publisher for GitHub Actions with owner `simkimsia`, the repo name, and workflow filename `release-please.yml`.
The package must already exist on npm, which is why 0.1.0 was published by hand.

Why the `paths-ignore` blocks: release-please opens its PR with `GITHUB_TOKEN`, and a `pull_request` run triggered by `GITHUB_TOKEN` sits in `action_required` and never starts.
That PR only touches the three generated files, so ignoring those paths means no stuck run is ever created.
The guard's author check alone cannot do this, because it is evaluated inside a run that never starts.

After the rollout merges in a repo, stop publishing that repo by hand.
