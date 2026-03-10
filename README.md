# Common Repository Procedures and Utilities

This contains a unified list of properties that repositories I manage should have. Useful for validating that everything is properly set up when dealing with a large number of repositories.

This repository also contains shared configuration files that other repos can reference:
- **Renovate**: Repos extend this config via `"extends": ["github>cwfitzgerald/repo-common"]`
- **deny.toml**: Template for cargo-deny configuration
- **pull_request_template.md**: Standard PR template
- **CHANGELOG.md**: Template for changelog format
- **RELEASE.md**: Template for release process

## Checklist

- [ ] Basic CI
  - [ ] CI_RUST_VERSION env var pinning the tested Rust version
  - [ ] Using -Dwarnings for both RUSTFLAGS and RUSTDOCFLAGS
  - [ ] clippy
  - [ ] cargo doc --no-deps
  - [ ] cargo nextest run
  - [ ] cargo test --doc (not applicable to FFI crates)
  - [ ] cargo fmt --check
  - [ ] cargo deny check (with --all-features)
  - [ ] MSRV check with cargo check (allow warnings, test with and without default features)
  - [ ] If platform specifics aren't important, only target windows + mac + linux + wasm. If they are, make sure to build (not just clippy) for both aarch64 and x86. Use rosetta on mac, linux aarch64 runners, and don't run tests on x86.
- [ ] Pinning
  - [ ] Rust version is pinned in rust-toolchain.toml
  - [ ] Any external binaries (e.g. ispc) use pinned versions in CI from an env var
  - [ ] Cargo.lock is committed
- [ ] Crates
  - [ ] rust-version field is specified and agrees with MSRV
  - [ ] All descriptive fields are specified (description, repository, readme, license, keywords, categories)
- [ ] deny.toml
  - [ ] Licenses allow-list configured (start from template, trim per-repo)
  - [ ] multiple-versions = "deny" with skip list as needed
  - [ ] unknown-registry = "deny"
  - [ ] advisories ignore list with comments explaining each entry
- [ ] Renovate
  - [ ] renovate.json extends `github>cwfitzgerald/repo-common`
- [ ] README
  - [ ] Documents MSRV and MSRV bump policy
  - [ ] Documents how to use the library/binary
- [ ] CHANGELOG.md
  - [ ] Follows Keep a Changelog format (see template)
  - [ ] Has table of contents and diff links at bottom
- [ ] RELEASE.md
  - [ ] Human and agent readable todo list for pushing releases forward
- [ ] Pull Request Template
  - [ ] .github/pull_request_template.md present
