# LEZ Atomic Swaps — Logos module catalog

This fork of `logos-co/logos-modules-release-base` publishes the Logos
packages of [gateway-fm/lez-atomic-swaps](https://github.com/gateway-fm/lez-atomic-swaps):
the Basecamp apps **LEZ / BTC Maker** (`lez_atomic_swap_maker`) and
**LEZ / BTC Taker** (`lez_atomic_swap_taker`) and, from v0.3.1, the packages
they and the LEZ network desk install. *Published packages and variants*
below lists each package and the platforms it ships for.

Add it to Basecamp (Settings → Repositories) with this catalog URL:

```
https://raw.githubusercontent.com/gateway-fm/logos-modules-release-base/main/logos-repo.json
```

Both apps live in one submodule (`submodules/lez-atomic-swaps`, under
`apps/basecamp/<role>`), so the per-module workflows point `module_path`
below the submodule root and `modules.txt` lists the module paths that
**Release all modules** fans out over. Each app carries its own `flake.nix`
next to its `metadata.json`; the app version in `metadata.json` follows the
upstream release tag (`0.2.1` ↔ `v0.2.1`). To publish a new version, bump the
submodule pointer to the release tag and run **Release all modules**:

```sh
git -C submodules/lez-atomic-swaps fetch --tags && git -C submodules/lez-atomic-swaps checkout v0.2.1
git add submodules/lez-atomic-swaps && git commit -m "lez-atomic-swaps v0.2.1" && git push
./scripts/catalog.sh release-all --watch
```

The v0.3.1 packages are signed; see *Package signatures* below.

## Package signatures

The eight v0.3.1 packages (the table below) are signed with one Ed25519
key. Its identity, also listed under `trustedSigners` in `logos-repo.json`:

```
did:jwk:eyJjcnYiOiJFZDI1NTE5Iiwia3R5IjoiT0tQIiwieCI6Ik42REI2QWlOOTJZSnRzMDdLa1NfV3k5MGVqWnp4QlNTWjdPQ2FHZEd4aEkifQ
```

A signature covers the package's `manifest.json`, and the manifest's
`hashes` cover every file in the package, so a changed file or a changed
manifest no longer verifies. Check a downloaded package with `lgx`
(logos-package):

```sh
lgx verify lez_atomic_swap_maker-0.3.1.lgx
```

It prints `Signature is valid` and the signer DID above. To trust the key,
add it to your keyring once:

```sh
lgx keyring add lez-atomic-swaps did:jwk:eyJjcnYiOiJFZDI1NTE5Iiwia3R5IjoiT0tQIiwieCI6Ik42REI2QWlOOTJZSnRzMDdLa1NfV3k5MGVqWnp4QlNTWjdPQ2FHZEd4aEkifQ
logosctl key add lez-atomic-swaps --did did:jwk:eyJjcnYiOiJFZDI1NTE5Iiwia3R5IjoiT0tQIiwieCI6Ik42REI2QWlOOTJZSnRzMDdLa1NfV3k5MGVqWnp4QlNTWjdPQ2FHZEd4aEkifQ
```

`lgx keyring` writes `~/.config/logos/trusted-keys/`; logosctl 0.3.2 keeps
its own keyring in its session directory (`<config-dir>/keyring`), so add
the key with `logosctl key add` for logosctl. Without the key in the
keyring, logosctl 0.3.2 installs a package and logs
`Warning: Package is signed by a key no trusted anchor validates`, with the
signer DID; with it, the install logs no warning. A package whose signature
does not verify is refused (`Invalid signature: Ed25519 signature
verification failed`).

`trustedSigners` is advisory in logosctl and Basecamp 0.3.2: both read the
field, neither trusts a key because it is listed there. Only the local
keyring decides whether a signer is trusted.

Basecamp 0.3.2 still words its install prompt as “unsigned” for every
third-party package (“not part of Logos, and unsigned, so the shell cannot
confirm who published it”), so it shows that text for these signed
packages too, with or without the key in the keyring.

## Published packages and variants

Each release carries one `.lgx` that holds every variant of that package.
The v0.3.1 set:

| Package | Release | `darwin-arm64` | `linux-amd64` | `linux-arm64` |
|---|---|:-:|:-:|:-:|
| `lez_atomic_swap_maker` | `lez_atomic_swap_maker-v0.3.1` | yes | yes | yes |
| `lez_atomic_swap_taker` | `lez_atomic_swap_taker-v0.3.1` | yes | yes | yes |
| `lez_swap_node` | `lez_swap_node-v0.3.1` | yes | yes | yes |
| `lez_maker_node` | `lez_maker_node-v0.3.1` | yes | yes | yes |
| `lez_taker_node` | `lez_taker_node-v0.3.1` | yes | yes | yes |
| `lez_core` | `lez_core-v0.4.1-sign.4` | yes | yes | yes |
| `lez_network` | `lez_network-v0.3.1` | no | yes | yes |
| `lez_indexer_module` | `lez_indexer_module-v1.1.1-rpc.4` | no | yes | yes |

`lez_network` (the LEZ network's operator desk) and `lez_indexer_module`
are Linux-only. The earlier desk releases, `lez_atomic_swap_maker` and
`lez_atomic_swap_taker` v0.2.1 to v0.2.5, carry `darwin-arm64`,
`linux-amd64` and `linux-arm64`. No package in this catalog has a Windows
variant.

On an x86_64 Linux host the client reports its platform as
`linux-x86_64` and installs the `linux-amd64` variant (logosctl 0.3.2).
The v0.3.1 macOS executables are signed ad hoc, not with a Developer ID, and are
not notarised.

The v0.3.1 packages are built and merged by the `release-packages.yml`
workflow of gateway-fm/lez-atomic-swaps and attached to these releases;
this catalog's own release workflow builds only the two desk modules in
`modules.txt`.

---

# logos-modules-release-base

A **fork-me** starting point for running your own Logos module catalog.

Fork this repo, add your modules as submodules, push — and you have a
working module repository that the Logos clients (`lgpd`, the
`package_downloader` module, the package-manager UI) can install from.

All the release machinery comes from the reusable
[`logos-co/logos-modules-release-action`](https://github.com/logos-co/logos-modules-release-action).
This catalog carries a vendored copy of its v1 workflows under
`.github/workflows/_logos-*.yml` (see [`.github/VENDORED.md`](.github/VENDORED.md)),
next to *your* submodules, *your* catalog metadata, and a
thin layer of workflows that call the action. The canonical production
catalog,
[`logos-co/logos-modules-v2`](https://github.com/logos-co/logos-modules-v2),
is built exactly this way.

## Quick start

1. **Fork** this repo (GitHub → *Use this template* / *Fork*).

2. **Edit `logos-repo.json`** — this is how clients identify your
   catalog. Replace every `CHANGE-ME` and set `indexUrl` to point at
   *your* fork:

   ```json
   {
     "schemaVersion": 1,
     "name": "my-modules",
     "displayName": "My Modules",
     "description": "My personal Logos modules.",
     "homepage": "https://example.com",
     "indexUrl": "https://github.com/<your-owner>/<your-repo>/releases/download/index/index.json",
     "trustedSigners": []
   }
   ```

3. **Add your modules**:

   ```bash
   git clone https://github.com/<your-owner>/<your-repo>
   cd <your-repo>
   ./scripts/add-module.sh https://github.com/<you>/<your-module-repo>
   git add -A && git commit -m "Add <your-module-repo>" && git push
   ```

   `add-module.sh` registers the submodule **and** generates its
   per-module release workflow. Repeat for each module.

4. **Publish.** From the repo's **Actions** tab, run **Release all
   modules** (or an individual **Release \<module\>**) — or, from a
   terminal, `./scripts/catalog.sh release-all`. The action builds each
   `.lgx`, verifies it, optionally signs it, cuts a `<module>-v<version>`
   GitHub release, and rolls everything up into the `index` release that
   clients read.

5. **Point a client at it.** Add your fork's `logos-repo.json` raw URL
   as a repository in the package-manager UI / `lgpd`:

   ```
   https://raw.githubusercontent.com/<your-owner>/<your-repo>/<default-branch>/logos-repo.json
   ```

That's it. Bumping a submodule pointer (which moves its
`metadata.json#version`) and re-running its workflow publishes a new
version; clients pick it up on their next catalog refresh.

## Layout

```
.
├── logos-repo.json                       # YOUR catalog metadata — edit this
├── .gitmodules                           # submodule declarations (starts empty)
├── submodules/                           # one git submodule per module (you add these)
├── scripts/
│   ├── add-module.sh                     # add a submodule + generate its workflow
│   └── catalog.sh                        # run the catalog workflows via `gh` (no Actions tab)
└── .github/workflows/
    ├── _release-module.yml               # signing config — the ONE place to edit it
    ├── release-module.yml.template       # per-module workflow template (don't run; it's a template)
    ├── release-all.yml                   # umbrella; discovers modules from .gitmodules
    ├── rebuild-index.yml                 # rebuilds index.json after each release
    └── unpublish.yml                     # manually remove a module / version from the catalog
```

### Workflow architecture

Two-tier reusable workflows so the signing pipeline lives in exactly
one place:

- **`_release-module.yml`** — local *private* reusable workflow
  (`workflow_call` only, so it never shows up as runnable in the
  Actions UI). Calls `./.github/workflows/_logos-release.yml` — the
  vendored copy of
  `logos-co/logos-modules-release-action/.github/workflows/release.yml` at v1 —
  with this catalog's signing configuration and variant list.
- **`release-<module>.yml`** — one per module, generated by
  `add-module.sh`. Each just passes `module_path: submodules/<repo>` to
  `_release-module.yml`. Lets you cut a single module from the Actions
  UI.
- **`release-all.yml`** — umbrella. **Discovers** the module list from
  `.gitmodules` at run time (no hand-maintained matrix) and fans out to
  `_release-module.yml` per module in parallel.
- **`rebuild-index.yml`** — thin passthrough to the action's
  index-rebuilder. Auto-triggered after each release; also runs on a
  6-hourly catch-up schedule.
- **`unpublish.yml`** — manual (Actions tab). Removes a whole module
  or one specific version: deletes the release(s) + optionally their
  tags, then rebuilds the index. **Run with `dry_run: true` first** —
  deletion is irreversible. Re-running a release for an unchanged
  submodule is a fast no-op (the action skips builds whose
  `<module>-v<version>` is already published).

Both **Release \<module\>** and **Release all modules** take a **Force
build** toggle on the *Run workflow* form (off by default). Leave it off
for the normal skip-if-already-published behaviour; turn it on to rebuild
and **replace** the current release when the version is unchanged — handy
after a half-published release or a build-pipeline fix. From the terminal
it's `./scripts/catalog.sh release <module> --force` (or `release-all
--force`).

Every workflow declares `permissions: contents: write` because a
forked repo's default `GITHUB_TOKEN` is read-only, and the release job
must create releases / upload assets.

To change signing or the variant list for the whole catalog, edit
**`_release-module.yml` only** — the per-module callers and the umbrella
never need touching.

## Signing

Ships **unsigned** (`signing_mode: none`) so a fresh fork works
immediately. When you're ready to sign, follow the inline instructions
at the top of
[`.github/workflows/_release-module.yml`](.github/workflows/_release-module.yml):

| Mode | What runs | Where the key lives |
|---|---|---|
| `none` (default) | nothing | n/a — unsigned releases |
| `inline` | `lgx sign` in CI | `LOGOS_SIGNING_KEY` Actions secret (Ed25519 JWK) |
| `external` | your `signing_command` | anywhere (Jenkins / HSM / hardware token) |

For `inline`, also put the matching public DID under `trustedSigners`
in `logos-repo.json`; logosctl and Basecamp 0.3.2 treat that list as
advisory (see *Package signatures*).

The release workflows here still run with `none`. The eight v0.3.1
packages were signed with `lgx sign` after their release builds, and the
signed files replaced the release assets.

## Variants

What this catalog actually ships, per package, is in *Published packages
and variants* at the top of this file.

The release workflow builds the variants named on the `variants:` line of
`_release-module.yml`. In this catalog that line is
`darwin-arm64,linux-amd64,linux-arm64`:

| Variant | Runner | Built from |
|---|---|---|
| `darwin-arm64` | `macos-latest` | native |
| `linux-amd64` | `ubuntu-latest` | native |
| `linux-arm64` | `ubuntu-24.04-arm` | native |

The upstream template also offers `windows-x86_64`, cross-compiled with
mingw on a Linux runner for a module whose flake exposes
`packages.x86_64-windows`; this catalog does not build it. A release that
ships fewer variants than requested records which in its sidecar's
`builtVariants` / `missingVariants`.

## Pinning the action version

The action's workflows are vendored in this repository at a fixed
upstream commit, so releases do not follow the upstream `v1` tag.
[`.github/VENDORED.md`](.github/VENDORED.md) records the commit and the
update procedure. Upstream `v2` signals a breaking change to the
workflow inputs or the index schema; read its migration notes before
vendoring it.

## Managing `index.json` without GitHub Actions

If you'd rather host `.lgx` files yourself (S3, your own server, a
file share — anywhere `lgpd` can `GET` from) and not rely on this
repo's GitHub-Actions index builder,
[`logos-co/logos-modules-release-tool`](https://github.com/logos-co/logos-modules-release-tool)
ships a single-file CLI (`index.py`) that builds, edits and validates
an `index.json` from a plain list of `.lgx` URLs (or local files for
packages you've just uploaded yourself). The output is byte-compatible
with what `rebuild-index.yml` here produces, so clients consume it
identically.

The GitHub Actions path in this repo stays the default — the tool is
for catalog maintainers who want a fully non-GitHub setup, or who want
local control over partial / incremental index edits.

## Notes for cloning a fork

`.gitmodules` is committed but submodule working trees are not — after
cloning your fork, run:

```bash
git submodule update --init --recursive
```
