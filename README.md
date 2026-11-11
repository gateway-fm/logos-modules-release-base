# LEZ Atomic Swaps — Logos module catalog

This fork of `logos-co/logos-modules-release-base` publishes the two Basecamp
apps of [gateway-fm/lez-atomic-swaps](https://github.com/gateway-fm/lez-atomic-swaps):
**LEZ / BTC Maker** (`lez_atomic_swap_maker`) and **LEZ / BTC Taker**
(`lez_atomic_swap_taker`).

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

Releases are unsigned (`trustedSigners` is empty); see *Signing* below.

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
in `logos-repo.json` so clients trust your signature.

## Variants

This catalog publishes four variants:

| Variant | Runner | Built from |
|---|---|---|
| `darwin-arm64` | `macos-latest` | native |
| `linux-amd64` | `ubuntu-latest` | native |
| `linux-arm64` | `ubuntu-24.04-arm` | native |
| `windows-x86_64` | `ubuntu-latest` | mingw cross |

Windows is the odd one: it is **cross-compiled**, so a Linux runner
produces the PE, and only a module whose flake exposes
`packages.x86_64-windows` has anything to build there. A module that
never opted into the Windows target fails that one leg and still ships
the other three as a partial release — the sidecar records which in
`builtVariants` / `missingVariants`.

Drop `windows-x86_64` from the `variants:` line in
`_release-module.yml` if you'd rather not see a red leg on modules that
don't support it.

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
