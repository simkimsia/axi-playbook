# Distribution

None of the four repos is on npm yet.
They install from a clone today, and this doc covers how to get from there to a release without surprises.

## Check the npm name before naming the repo

The SDK's built-in `update` command resolves the package by its npm name from `package.json`.
If someone else owns that name on npm, `<vendor>-axi update` installs their package.

This has already happened.
The npm name `cloudflare-axi` belongs to a different author (radityasurya), so `update` in my cloudflare-axi would resolve to their package.

Before creating a new repo:

```sh
npm view <vendor>-axi name    # E404 means the name is free
```

The convention I settled on: publish every axi under my npm scope, `@simkimsia/<vendor>-axi`, and keep the command name unscoped through `bin` (`"bin": { "<vendor>-axi": ... }`).
A scope I own cannot be taken by someone else, and using it for all four keeps them consistent even where the unscoped name is free.

Status as of 2026-10-04: the rename is prepared on a `chore/scoped-npm-name` branch in each of the four repos and is not on `main` yet.
On `main`, `package.json` still has the unscoped name.

If a repo ever has to keep an unscoped name, the SDK's `packageName` option on `runAxiCli` overrides the name `update` resolves.
Decide before the first release, because changing the name later breaks every existing install's `update`.

## Install from a clone

Until a release exists, the README and the skill give this install:

```sh
git clone https://github.com/simkimsia/<vendor>-axi
pnpm --prefix <vendor>-axi install
pnpm --prefix <vendor>-axi run build
pnpm add -g link:$PWD/<vendor>-axi   # puts `<vendor>-axi` on PATH
```

Do not use `pnpm --prefix <vendor>-axi link --global`.
With pnpm 10.33, `link --global` ignores `--prefix` and links the current directory instead, so the axi never lands on PATH.

This was fixed in railway-axi ([#5](https://github.com/simkimsia/railway-axi/issues/5)) and is still open in [cloudflare-axi#12](https://github.com/simkimsia/cloudflare-axi/issues/12), [netlify-axi#1](https://github.com/simkimsia/netlify-axi/issues/1), and [calcom-axi#1](https://github.com/simkimsia/calcom-axi/issues/1).

The README also says plainly that `npx -y @simkimsia/<vendor>-axi` does not work yet, so nobody tries it and gets someone else's package.

## Release plan: release-please

Not adopted in any repo yet. The model is gh-axi:

- [release-please.yml](https://github.com/kunchenguid/gh-axi/blob/main/.github/workflows/release-please.yml) opens a release PR from conventional commits on `main`. When that PR merges, the same workflow builds and runs `npm publish --access public --provenance`.
- [guard-generated-files.yml](https://github.com/kunchenguid/gh-axi/blob/main/.github/workflows/guard-generated-files.yml) fails any human PR that edits `CHANGELOG.md` or `.release-please-manifest.json`, because release-please owns them.

Why release-please: versions and changelogs come from commit messages already written in conventional form ([process.md](process.md)), and publishing with provenance ties each npm version to the commit and workflow that built it.

Order of work for the first release in each repo:

1. Merge the scoped npm name (above).
2. Add the two workflows and an `NPM_TOKEN` or trusted publisher.
3. Change the README and skill install to `npx -y @simkimsia/<vendor>-axi` plus the global install.
