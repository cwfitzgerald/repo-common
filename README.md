# Common Repository Procedures and Utilities

This contains a unified list of properties that repositories I manage should have. Useful for validating that everything is properly set up when dealing with a large number of repositories.

This repository also contains shared configuration files that other repos can reference:
- **Renovate**: Repos extend this config via `"extends": ["github>cwfitzgerald/repo-common"]`
- **Install Rust action**: Shared composite action at `.github/actions/install-rust`
- **deny.toml**: Template for cargo-deny configuration
- **pull_request_template.md**: Standard PR template
- **CHANGELOG.md**: Template for changelog format
- **RELEASE.md**: Template for release process

## Shared GitHub Action

### Install Rust

Usage:
```yaml
env:
  CI_RUST_VERSION: "1.94"

steps:
  - uses: cwfitzgerald/repo-common/.github/actions/install-rust@trunk
    with:
      version: ${{ env.CI_RUST_VERSION }}
      components: clippy, rustfmt  # optional
      targets: x86_64-unknown-linux-gnu  # optional
```

The `version` input is required. Uses `rustup toolchain install` with minimal profile.

## Checklist

### CI

CI should generally have 4 jobs: the main CI matrix, format, deny, and MSRV.
Keep existing workflow filenames, job IDs, and display names (`name:`) — GitHub
uses display names to match required status checks in branch protection rules,
so renaming them will break merge queues. When splitting a single job into
multiple jobs, keep the original display name on whichever job replaces it and
pick new names for the added jobs.
Do not downgrade action versions (e.g. checkout@v6 to checkout@v4) without asking.
Make only minimal changes to CI to make things compliant. If there is a conflict, ask.
Don't make major structural changes to existing CI.

- [ ] `CI_RUST_VERSION` env var pinning the tested Rust version
- [ ] `CI_RUST_MSRV` env var pinning the MSRV
- [ ] Uses `cwfitzgerald/repo-common/.github/actions/install-rust@trunk` to install Rust
- [ ] Using `-Dwarnings` for both `RUSTFLAGS` and `RUSTDOCFLAGS`
- [ ] **CI job** (platform matrix):
  - [ ] `cargo-nextest` installed via `taiki-e/install-action@cargo-nextest`
  - [ ] `cargo clippy --all-features`
  - [ ] `cargo build` (only needed for crates with native/FFI dependencies where linking matters)
  - [ ] `cargo doc --no-deps`
  - [ ] `cargo nextest run`
  - [ ] `cargo test --doc` (not applicable to crates exposing a C interface)
  - [ ] If platform specifics aren't important, only target windows + mac + linux + wasm. If they are, make sure to build (not just clippy) for both aarch64 and x86. Use rosetta on mac, linux aarch64 runners, and don't run tests on windows aarch64.
- [ ] **Format job**: `cargo fmt --check`
- [ ] **Deny job**: `cargo deny --all-features check`
- [ ] **MSRV job**: `cargo check` with and without default features (clear `RUSTFLAGS`/`RUSTDOCFLAGS` to allow warnings)

### Pinning

- [ ] Rust version is pinned in `rust-toolchain.toml`
- [ ] Any external binaries (e.g. ispc) use pinned versions in CI from an env var
- [ ] `Cargo.lock` is committed

### Crates

- [ ] `rust-version` field is specified and agrees with MSRV
- [ ] All descriptive fields are specified (`description`, `repository`, `readme`, `license`, `keywords`, `categories`)

### deny.toml

- [ ] Licenses allow-list configured (start from template, trim unused licenses)
- [ ] `multiple-versions = "deny"` with skip list as needed (each entry should have a `reason`)
- [ ] `unknown-registry = "deny"`
- [ ] Advisories ignore list with comments explaining each entry

### Renovate

- [ ] `renovate.json` extends `github>cwfitzgerald/repo-common`

### README

- [ ] Documents MSRV and MSRV bump policy
- [ ] Documents how to use the library/binary

### CHANGELOG.md

- [ ] Follows Keep a Changelog format (see template)
- [ ] Has table of contents and diff links at bottom

### RELEASE.md

- [ ] Human and agent readable todo list for pushing releases forward

### Pull Request Template

- [ ] `.github/pull_request_template.md` present

## Verification Commands

After setting up a new repo (or auditing an existing one), the following commands should all pass locally:

```bash
# Lint
cargo clippy --all-targets -- -D warnings

# Format
cargo fmt --check

# Doc (should produce no warnings)
RUSTDOCFLAGS="-D warnings" cargo doc --no-deps

# Dependency checks
cargo deny --all-features check

# Tests
cargo nextest run
cargo test --doc

# MSRV (replace 1.XX with your MSRV from Cargo.toml rust-version)
# These should compile without errors; warnings are allowed
rustup run 1.XX cargo check
rustup run 1.XX cargo check --no-default-features
```

## Agent Prompts

### Audit a project against this standard

```
Read the checklist and templates in the repo-common repository at ../repo-common
(README.md, deny.toml, pull_request_template.md, CHANGELOG.template.md,
RELEASE.template.md, .github/actions/install-rust/action.yml).

Then audit the current repository against every item in the checklist. For each
item, report whether it passes, fails, or is not applicable, with a brief
explanation. At the end, summarize what needs to change.

Do not make any changes — this is a read-only review.
```

### Apply this standard to a project

```
Read the checklist and templates in the repo-common repository at ../repo-common
(README.md, deny.toml, pull_request_template.md, CHANGELOG.template.md,
RELEASE.template.md, .github/actions/install-rust/action.yml).

Apply every item in the checklist to the current repository:

1. Create or update the CI workflow with the 4-job structure
   (ci matrix, format, deny, MSRV). Use the shared install-rust action. Pin
   the Rust version via CI_RUST_VERSION and MSRV via CI_RUST_MSRV. Use
   cargo-msrv or edition requirements to determine the MSRV if not already set.
2. Create rust-toolchain.toml pinning the CI Rust version.
3. Copy deny.toml from the template, then run `cargo deny --all-features check`
   and fix all failures by adjusting the allow-list and skip-list. Trim any
   licenses from the allow-list that produce "not encountered" warnings.
4. Ensure all workspace and crate Cargo.toml files have: rust-version,
   description, repository, readme, license, keywords, categories.
5. Set renovate.json to extend github>cwfitzgerald/repo-common.
6. Add an MSRV section to README.md stating the MSRV and that bumps are
   breaking changes.
7. Create CHANGELOG.md, RELEASE.md, and .github/pull_request_template.md
   from the templates, substituting crate/repo names as needed.
8. Run the verification commands and fix any issues.

Ask questions for anything ambiguous before making changes.
```
