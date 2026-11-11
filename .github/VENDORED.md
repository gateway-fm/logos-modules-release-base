# Vendored GitHub Actions code

The organisation requires every external `uses:` to be a full commit SHA,
including those nested in reusable workflows and composite actions. Upstream
references tags, so the code is copied here and called by local path.

| Here | Source | Upstream commit | Licence |
| --- | --- | --- | --- |
| `.github/workflows/_logos-release.yml` | [`logos-co/logos-modules-release-action`](https://github.com/logos-co/logos-modules-release-action) `.github/workflows/release.yml` | `bbdc52871dc4305c8007923e1b08738ad2dd6b50` (v1) | MIT |
| `.github/workflows/_logos-unpublish.yml` | same repository, `.github/workflows/unpublish.yml` | same | MIT |
| `.github/workflows/_logos-rebuild-index.yml` | same repository, `.github/workflows/rebuild-index.yml` | same | MIT |
| `.github/actions/setup-nix-cache/` | [`logos-co/setup-nix-cache-action`](https://github.com/logos-co/setup-nix-cache-action) `action.yml` | `781f072122f4df6af0e33a7d355ebd20283e019e` (v1.2.0) | MIT OR Apache-2.0 |

`logos-modules-release-action` states its licence as "MIT." in its README and
ships no LICENSE file or copyright line. `setup-nix-cache-action` ships
`LICENSE-MIT` (Copyright © 2025-2026 Logos) and `LICENSE-APACHE`; both are
copied unchanged beside `action.yml`.

## Changes from upstream

- `cachix/install-nix-action@v31` and `ryanccn/attic-action@v0.5.0` in
  `action.yml` are pinned to commit SHAs.
- `logos-co/setup-nix-cache-action@781f072…` in the three workflows is replaced by
  `./.github/actions/setup-nix-cache`. Every job that uses it runs
  `actions/checkout` of this repository first.
- `_logos-rebuild-index.yml`: the `workflow_dispatch`, `repository_dispatch`
  and `schedule` triggers are removed. `rebuild-index.yml` owns them; left in,
  each event would start two rebuilds in the same concurrency group.
- `_logos-unpublish.yml`: the `workflow_dispatch` trigger is removed.
  `unpublish.yml` is the one manual entry point.
- A three-line provenance header is added to each file.

Nothing else differs. The workflows reference no other file of their own
repository. `_logos-rebuild-index.yml` checks out
`logos-co/logos-modules-release-tool` at `v1.3` at run time, as upstream does.

## Updating from upstream

1. Copy the three workflows and `action.yml` from the new upstream commits
   over the files above.
2. Reapply the changes listed in "Changes from upstream"; pin every external
   `uses:` to a 40-hex SHA with the tag as a trailing comment.
3. Record the new commits in the table, run `actionlint`, and run
   "Rebuild index" on the branch before merging.
