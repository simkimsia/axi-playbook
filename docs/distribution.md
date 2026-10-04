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

Only the first version of a package is published by hand, because npm's trusted publisher setting needs the package to exist already.
After that, releases go through release-please (below).
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

Status: live in railway-axi, netlify-axi, cloudflare-axi and calcom-axi since 2026-10-04.
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
Trusted publishing needs npm 11.5.1 or later, and the npm bundled with the runner's Node 24 can be older, so the workflow runs `npm install -g npm@11` before publishing.
It pins 11 rather than `latest` because npm 12 refuses to run on older Node 24 minors (it needs `^24.15.0`), so `latest` can break the job on a runner image update.
Provenance ties each npm version to the commit and workflow run that built it.

Version bumps below 1.0: `feat` and `fix` bump the patch, and a breaking change (`!` or a `BREAKING CHANGE:` footer) bumps the minor.
That is why cloudflare-axi's first automated release was 0.2.0, not 0.1.1: it carried the `NOT_CONFIGURED` to `NOT_LINKED` rename.

Why the `paths-ignore` blocks: release-please opens its PR with `GITHUB_TOKEN`, and a `pull_request` run triggered by `GITHUB_TOKEN` sits in `action_required` and never starts.
That PR only touches the three generated files, so ignoring those paths means no stuck run is ever created.
The guard's author check alone cannot do this, because it is evaluated inside a run that never starts.

The changelog has one author, release-please.
It writes `CHANGELOG.md` and the GitHub Release notes from the same commits, and the guard keeps hand edits out, so there is no hand-kept "Unreleased" section like gh-axi's.
`CHANGELOG.md` is in the package's `"files"` list ([repo-skeleton.md](repo-skeleton.md#packagejson)), so an installed copy carries its own release notes, including breaking error-code renames an agent may hit after `update`.
The README links it under `## Changelog` ([template](../templates/README.md)).

After the rollout merges in a repo, stop publishing that repo by hand.

### One-time setup per repo

The workflow file is not enough on its own.
Two settings live outside the repo, one on npm and one on GitHub, and each failed the first time it was missed.
Do both before merging the PR that adds release-please.

**1. npm: add the trusted publisher.**
It is a per-package setting, not an account setting.
Open the package page, for example `https://www.npmjs.com/package/@simkimsia/railway-axi`, and click the **Settings** tab, which only shows when you are logged in as a maintainer.
The direct link is `https://www.npmjs.com/package/@simkimsia/<vendor>-axi/access`.
Under **Trusted Publisher**, choose GitHub Actions and enter owner `simkimsia`, the repo name, and workflow filename `release-please.yml`.
Leave the environment empty.

- Check the permissions on the saved entry.
  npm separates **publish** from **stage publish**, and an entry can end up with stage publish only.
  The workflow runs a plain `npm publish`, so the entry must allow publish.
  Fix it with **Edit** on the entry.
- "a trusted publisher configuration that a token could also match already exists for this package" means the entry was already saved.
  It is not a failure; edit the existing entry instead of adding a second one.
- **Publishing access** on the same page does not affect trusted publishing; npm says so in a note under it.
  The recommended choice is "Require two-factor authentication and disallow bypass 2fa tokens", which blocks long-lived tokens that skip 2FA, the usual way npm packages get hijacked.
  Changing it does not fix a failed release.
- To check the entries from a terminal, `npm trust list @simkimsia/<vendor>-axi` exists from npm 12 (`npx -y npm@latest trust list ...`).
  It needs a one-time code, and in a non-interactive shell (an agent's shell, or a `!` command) the browser sign-in cannot complete, so pass `--otp=<code>`.
  If the account uses a passkey, check in the browser instead.

**2. GitHub: let Actions open pull requests.**
release-please opens its release PR with `GITHUB_TOKEN`.
New repos default to not allowing that, and the release-please job fails with:

```text
release-please failed: GitHub Actions is not permitted to create or approve pull requests.
```

The run had already pushed the release branch, so the only thing missing is the PR.
Turn the setting on and keep the default token read-only:

```sh
gh api -X PUT repos/simkimsia/<vendor>-axi/actions/permissions/workflow \
  -f default_workflow_permissions=read -F can_approve_pull_request_reviews=true
```

In the browser it is Settings, Actions, General, in the **Workflow permissions** box at the bottom of the page, not the "Actions permissions" radio buttons at the top: keep "Read repository contents and packages permissions" and tick "Allow GitHub Actions to create and approve pull requests".
The workflow still gets `contents: write`, `pull-requests: write` and `id-token: write` from its own `permissions:` block, so the read-only default holds for every other workflow.

Then rerun the latest failed release-please run rather than an older one, since release-please reads `main` as it is now:

```sh
gh run list -R simkimsia/<vendor>-axi --workflow release-please.yml --limit 1
gh run rerun <run-id> -R simkimsia/<vendor>-axi --failed
```

**3. First automated release.**
Merge one repo's release PR first and confirm the new version appears on npm before merging the others.
That is the first real test of the trusted publisher entry, and one failure is cheaper to read than four.
