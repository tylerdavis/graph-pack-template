# graph pack template

A starting point for a [graph](https://github.com/tylerdavis/graph) pack: a git repo of tools, plans and agents that people install with one command.

This template is a working pack named `example`. It describes files on disk: an exec tool reads a file's stats, a plan summarizes the file, and an agent answers questions using both. Copy the repo, rename the pack, and replace the contents with your own.

## Layout

```
pack.toml            manifest: name, description, requirements
tools/*.yaml         tool files (graph's tool format)
plans/*.yaml         plan files (graph's plan format)
agents/*.yaml        agent files (graph's agent format)
CHANGELOG.md         one entry per release tag
MIGRATING.md         what changed in each major and how to move to it
README.md            what the pack does and how to set it up
```

Every YAML file carries its own `version`, the graph file version it's written in, exactly as it would in a project's `.graph/` directory.

## Names

Everything in a pack is named `<pack>__<kind>__<name>`, where `<kind>` is `tool`, `plan` or `agent`:

| File | Catalog name |
|---|---|
| `tools/file_info.yaml` (`name: file_info`) | `example__tool__file_info` |
| `plans/describe_file.yaml` (`identifier: describe_file`) | `example__plan__describe_file` |
| `agents/file_helper.yaml` (`name: file_helper`) | `example__agent__file_helper` |

Plans and agents in the pack refer to each other by those full names. Plans can also call anything else in the user's catalog: `builtin__infer`, MCP tools, other packs.

## The manifest

```toml
version = 1
name = "example"
description = "Describe files on disk: stats, an excerpt, and a one-paragraph summary."
license = "MIT"
homepage = "https://code.home.davis.cool/tmd/pack-template"

[requires]
graph = ">=0.17"
os = ["macos", "linux"]
binaries = ["bash", "jq", "wc", "file", "head"]
env = ["EXAMPLE_TOKEN"]

[requires.mcp.linear]
server = "linear-mcp-server"
version = ">=1.0"
url = "https://mcp.linear.app/mcp"

[catalog]
hidden = ["tool__migrate_*", "plan__migrate_*"]
```

| Key | Checked | When it isn't met |
|---|---|---|
| `version` | on load | the manifest's file version; a version outside graph's support window is refused by name |
| `name` | on install | the default install name; must not be `user`, `plan`, `agent`, `builtin` or a configured MCP server name |
| `requires.graph` | on install | `graph pack add` refuses |
| `requires.os` | on install | `graph pack add` refuses |
| `requires.binaries` | install, `graph config check`, before a plan's first step | reported by name; the pack still installs |
| `requires.env` | install, `graph config check`, before a plan's first step | reported by name; the pack still installs |
| `requires.mcp.<prefix>` | first connection to that server | fails naming the difference |
| `catalog.hidden` | on load | globs over `<kind>__<name>` the pack keeps out of the catalog, tool search and the planner, the same way a user's `exclude` does; hidden items stay callable by full name |

`requires.mcp.<prefix>` is keyed by the tool prefix the pack's plans call (`linear__save_issue` needs `linear`), so the user's server must be configured under that name. On first connection graph checks the server's reported `serverInfo` name and version against `server` and `version`, and checks that every `linear__*` tool the pack's plans call exists with a compatible input schema. graph derives that tool list from the plans, so it isn't written in the manifest. `url` is a setup hint: `graph pack add` prints a config snippet for it and never writes config itself.

The example manifest above shows every key. This pack's real `pack.toml` uses only what it needs.

## Releases

A release is a semver git tag: `v1.0.0`, `v1.1.0`, `v2.0.0`. The tag is the pack's version, and the manifest doesn't repeat it.

- A **patch** fixes behavior without changing any name, input or output.
- A **minor** adds tools, plans, agents, optional inputs or output fields.
- A **major** renames or removes anything, adds a required input, or changes an output shape.

`graph pack update` moves installed packs to the newest tag within their current major. Crossing a major takes `graph pack update --major`, which runs the pack's migration for it.

## Major versions and migrations

Every major release ships a migration plan, `plans/migrate_v<N>.yaml`, and a `MIGRATING.md` section saying what changed. Keep the old migration plans when you add new ones: a user going from v1 to v3 runs `migrate_v2`, then `migrate_v3`, from the v3 files.

`graph pack update <pack> --major` runs each migration with this input, first with `apply: false`, shows the diffs, asks, then runs again with `apply: true`:

| Input | Type | Value |
|---|---|---|
| `pack` | string | the name the pack is installed under, which `--as` may have changed |
| `root` | string | the directory holding the user's `plans`, `agents` and `tools`, such as `.graph` or `~/.config/graph` |
| `apply` | boolean | `false` returns diffs only; `true` writes the files |

The plan's output is `{applied, files}`, each file `{path, changed, applied, diff}`. A pack that ships no migration plan for a major gets its `MIGRATING.md` printed instead.

This pack's `plans/migrate_v2.yaml` is the pattern: a scan tool finds files that mention the old names, a `map` asks `builtin__infer` to rewrite each one under exact rules, and a write tool returns the diff and writes only when `apply` is true. Running it twice changes nothing the second time. Its tools are listed in `catalog.hidden` so they never show up in tool search.

## Installing a pack

```bash
graph pack add github                                  # official: github.com/tylerdavis/graph-pack-github
graph pack add github.com/acme/graph-pack-example      # anyone else's, installed as "example"
graph pack add github.com/acme/graph-pack-example --as files
graph pack add example --global                        # into ~/.config/graph instead of the project
```

In a project, `add` copies the pack's files at the chosen tag into `.graph/packs/<name>/` and records it in config and the lockfile. Commit all three: CI runs from the committed copy and never fetches.

```toml
[packs.example]
git = "github.com/acme/graph-pack-example"
ref = "v1"
exclude = ["tool__file_info"]
```

`include` and `exclude` take globs over `<kind>__<name>`. They hide items from the catalog, tool search and the planner. Hidden items stay callable by their full name, so the pack's own plans keep working.

```toml
version = 1

[packs.example]
git = "github.com/acme/graph-pack-example"
tag = "v1.0.0"
sha = "4f1c2a9e0b7d3c5a8e6f1b2d4c7a9e0f3b5d8c1a"
hash = "sha256:9a1b…"
```

That's `.graph/packs.lock`. `hash` covers the copied files, so `graph pack update` refuses to overwrite local edits without `--force`, and `graph config check` reports a copy that no longer matches its lock.

```bash
graph pack list                  # installed tag, newest tag upstream, and what's behind
graph pack update                # every pack, within its current major
graph pack update example        # one pack
graph pack remove example
```

Only `list` and `update` touch the network.

## Checking a pack

```bash
graph pack check .
```

Loads the manifest and every file the way an install would, validates each plan and agent against the pack's own tools plus graph's built-ins, and reports requirements this machine doesn't meet. `.github/workflows/check.yaml` runs it on every push.
