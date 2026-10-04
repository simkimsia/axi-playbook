# Vision

`<vendor>-axi` is an agent-ergonomic interface to <Vendor>. It wraps the official CLI, `<cli>`, first, and calls <Vendor>'s public API only where `<cli>` has no surface or its output costs the agent extra calls.

## Scope

We aim for functional parity with `<cli>` on the surfaces agents operate: <surfaces>, and account identity.
Every capability available through `<cli>` should eventually be accessible through an AXI-native interface.

A command may call the public API when no `<cli>` command has the surface, or when one query replaces several CLI calls per row.
Every <Vendor> call reuses the credentials `<cli> login` already holds; we do not add separate token management.
We do not call <Vendor>'s MCP server; every MCP tool maps to an existing CLI command or public API operation, and keeping it out keeps it a separate arm in `bench/`.

The `bench/` directory, which measures `<vendor>-axi` against the raw `<cli>` CLI, is in scope and does not ship in the published package.

We accept contributions that expose existing <Vendor> capabilities more ergonomically.
We do not add functionality that <Vendor> itself does not provide, and we do not embed workflow logic that belongs in the calling agent.

## Interface

The interface must follow validated AXI principles and optimize for autonomous agent use.
These interface rules apply to commands; the `bench/` harness is not a command surface.

Output may be structured, but its structure exists for agent comprehension rather than as a stable API for imperative programs.
Human-oriented presentation and compatibility work primarily serving hand-written parsers are not goals.

Errors carry a stable code and a next step the agent can act on.
An unknown flag or argument is rejected by name before any `<cli>` call; it is never accepted silently.
The wrapper may reshape, combine, or simplify `<cli>` and API operations when doing so improves agent ergonomics without expanding the underlying capability.

## Safety

Read commands are the default and never change <Vendor> state.
Write commands are explicit, named as verbs, and print what changed, including any side effect they trigger.
A command that deletes or overwrites requires the target to be named in full; it never infers it from context.
When several resources could match and none is named with a flag or linked to the current directory, the command refuses and lists the names; it never guesses.
Secret values are printed only by a command that names one secret, and never appear in lists, errors, or logs.
