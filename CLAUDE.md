# vynkor-manager (`vynm`)

Plugin manager for the vynkor kernel: search/install/update/rollback plugins
from signed registries, write kernel drop-ins. See `README.md` for the CLI
and module map.

Shared process (branches, PR template, verification bar, merge order):
`../vynkor/docs/ENGINEERING_WORKFLOW.md`. Releases:
`../vynkor/docs/RELEASING.md`.

## Rules

- **Never depend on the `vynkor` kernel crate** — CI's C2 guard fails the
  build. Shared types come from `vynkor-wire`.
- Default branch `develop`; squash merges (`(#N)` in the subject).
- Gates: `cargo test --all-features`, `cargo clippy --all-targets
  --all-features -- -D warnings`, `cargo fmt --check`. Building needs
  `protoc` (vynkor-wire codegen).
- `Cargo.lock` is committed and releases build `--locked` — update it in
  the PR that changes dependencies.
- Paths are XDG and shared with the kernel: config
  `~/.config/vyn/config.yaml` (seeded by `vynm init`, also used by the
  kernel's `install.sh`), drop-ins `<config dir>/plugins.d/`, binaries
  `~/.local/lib/vyn/plugins/`, ledger `~/.local/share/vyn/installed.json`.
  Changing any of them is a cross-repo change (kernel, installer, site copy).
- Security boundaries (signature check against the pinned key, slug path
  validation, `create_new` drop-in writes) are ported verbatim — changes
  need a test that fails without the check.

## Releasing

Bump `version` in `Cargo.toml` → merge → `git tag vX.Y.Z` on `develop` →
`release.yml` publishes `vynm-<target>.tar.gz` (x86_64/aarch64 musl) +
`SHA256SUMS` + attestations. Release vynm **before** a kernel release that
depends on it. Dry run: `gh workflow run release.yml --ref <branch>`.
