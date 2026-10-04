# Distribution

All four repos are published on npm under my scope, as `@simkimsia/<vendor>-axi`.
Version 0.1.0 of each went out on 2026-10-04.
This doc covers the naming rule, the install paths, the publish gotchas I hit, and the release plan.

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

## Release plan: release-please

Not adopted in any repo yet. The model is gh-axi:

- [release-please.yml](https://github.com/kunchenguid/gh-axi/blob/main/.github/workflows/release-please.yml) opens a release PR from conventional commits on `main`. When that PR merges, the same workflow builds and runs `npm publish --access public --provenance`.
- [guard-generated-files.yml](https://github.com/kunchenguid/gh-axi/blob/main/.github/workflows/guard-generated-files.yml) fails any human PR that edits `CHANGELOG.md` or `.release-please-manifest.json`, because release-please owns them.

Why release-please: versions and changelogs come from commit messages already written in conventional form ([process.md](process.md)), and publishing with provenance ties each npm version to the commit and workflow that built it.

Order of work to adopt it in each repo:

1. Add the two workflows and an `NPM_TOKEN` or trusted publisher.
2. Seed `.release-please-manifest.json` with the published version, 0.1.0.
3. Stop publishing by hand.
