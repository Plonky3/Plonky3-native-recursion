# Contributing to Plonky3 Native Recursion

This is an experimental Plonky3 sibling workspace. Discuss substantial protocol,
API, or performance changes before implementing them so that the scope and
security assumptions are clear.

Use stable Rust for builds and Clippy, and nightly rustfmt. Start with the crate
you are changing:

```bash
cargo test -p p3-circuit --test tables
cargo test -p p3-circuit --test recursion_smoke
```

Before opening a pull request, run the routine workspace checks:

```bash
cargo +nightly fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
CARGO_PROFILE_TEST_OPT_LEVEL=1 cargo test --workspace
CARGO_PROFILE_TEST_OPT_LEVEL=1 cargo test --workspace --all-features
```

Ignored recursion tests need substantial memory and should be run individually
in release mode. See the [README](README.md#heavy-recursion-tests) for commands
and the memory estimates recorded in the source. State which checks you ran and
which heavy tests you did not run in your pull request.

Keep each pull request focused on one purpose. Describe the behavior changed and
why it is needed. Include regression coverage for bug fixes, explain protocol
and security assumptions near the affected public API, and give reproducible
measurements and machine details for performance changes. Avoid unrelated
refactors or generated result files.

Existing Plonky3 crates remain Git dependencies. Keep their upstream revision
consistent and check `cargo metadata` for duplicate package identities when
changing dependencies. The root patch for `p3-verifier-arith` must continue to
unify the arithmetic traits used by upstream verifiers and local circuits. See
the [dependency explanation](README.md#upstream-dependencies) before changing the
pin. Preserve original authorship and attribution when importing upstream work.

## Licensing

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed under MIT OR Apache-2.0, without any additional terms or
conditions.
