# <vendor>-axi

An [AXI](https://axi.md)-compliant wrapper around the [<Vendor>](https://<vendor-site>) CLI:
token-efficient [TOON](https://toonformat.dev) output, structured errors, and
agent-first ergonomics for AI coding agents that operate <Vendor> via shell.

Built on [`axi-sdk-js`](https://github.com/kunchenguid/axi), modeled on the
reference implementation [`gh-axi`](https://github.com/kunchenguid/gh-axi).

## Status

Early scaffold (v0). Read-only commands only.

## Requirements

- Node.js >= 20
- The [<Vendor> CLI](https://<vendor-cli-docs>) installed and logged in
  (`<cli> login`)

## Install

```sh
pnpm add -g @simkimsia/<vendor>-axi
```

Or run it without installing: `npx -y @simkimsia/<vendor>-axi --help`.

Check it: `<vendor>-axi --version`. Update later with `<vendor>-axi update`.

To work on it from a clone:

```sh
git clone https://github.com/simkimsia/<vendor>-axi
pnpm -C <vendor>-axi install
pnpm -C <vendor>-axi run build
pnpm add -g link:$PWD/<vendor>-axi   # puts `<vendor>-axi` on PATH
```

## Usage

```sh
<vendor>-axi            # dashboard: linked project, or recent items
<vendor>-axi whoami     # logged-in <Vendor> account
<vendor>-axi <command>  # <one line per command>
<vendor>-axi --help
<vendor>-axi --version  # fast path, never loads the command graph
<vendor>-axi update     # self-update (built into axi-sdk-js)
```

Example output (TOON):

```
count: <n> <items>
<items>[<n>]{<field>,<field>,<field>}:
  <row>
help[1]:
  Run `<vendor>-axi <command>` to <next step>
```

## Agent skill

Install the bundled skill so your coding agent prefers `<vendor>-axi` over raw
`<cli>`, falls back to `<cli>` when a command is not wrapped yet, and
files the gap as an issue here (label `agent-reported-gap`):

```sh
npx skills add simkimsia/<vendor>-axi --skill <vendor>-axi -g
```

The skill is a discovery stub that defers to `<vendor>-axi --help` for current
command guidance. Source: [`skills/<vendor>-axi/SKILL.md`](skills/<vendor>-axi/SKILL.md).

## Development

```sh
pnpm install
pnpm run dev          # run from source (tsx)
pnpm test             # vitest
pnpm run build        # tsc -> dist/
pnpm run format:check
```

## Changelog

Release notes live in [CHANGELOG.md](CHANGELOG.md) and on [GitHub Releases](https://github.com/simkimsia/<vendor>-axi/releases).
release-please writes both from conventional commits, so do not edit the file by hand.
Breaking changes, such as a renamed error code, are listed under "⚠ BREAKING CHANGES" and bump the minor version while below 1.0.

## License

MIT
