# Miyu package source

[中文](README.md)

The official package source of Miyu. The index is published at <https://miyu-agent.github.io/packages/>, and package files live in each package's own GitHub Releases. `miyu pkg update`, `miyu pkg search` and `miyu pkg install <id>` read from here.

## What is in this repository

| File | What it does |
|---|---|
| `sources.toml` | Which kinds of sources to collect. Only official repositories for now |
| `official.toml` | Official repositories. The `.miyu` files of each one's latest release are collected |
| `names.toml` | Package id to source repository. First come, first served, and the id stays with that repository. Collection writes new ids back |
| `blocked.toml` | Blocked ids or repositories, with a reason |
| `.github/workflows/collect.yml` | Collects once a day, signs the index and publishes it to Pages. Also runs when the rules change or by hand |
| `.github/workflows/pack.yml` | The packing template for package authors |

The index is signed with Ed25519. The public key is built into `miyu` (key id `f9234109a4740be9`). `miyu` rejects an index whose signature does not verify.

## Publishing a package

Add `.github/workflows/release.yml` to the package's repository:

```yaml
name: release
on:
  push:
    tags: ["v*"]
jobs:
  pack:
    uses: miyu-agent/packages/.github/workflows/pack.yml@main
    permissions:
      contents: write
    with:
      package: <package id>
```

Push a tag (`v` followed by the `version` in `package.toml`). The template checks the manifest the way Miyu checks it when installing, packs it into a `.miyu` file and uploads it to that tag's release.

- If the package folder is not the repository root, set `folder`.
- If programs must be built first, set `build` to a bash script.
- If the manifest lists `targets`, make one package file per target: use `strategy.matrix` with `runs-on` and `target`.

Only official repositories are collected for now. Automatic collection of community packages comes later.
