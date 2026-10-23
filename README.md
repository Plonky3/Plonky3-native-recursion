# Plonky3 Native Recursion

Experimental native recursion and hash-based signature aggregation built on
[Plonky3](https://github.com/Plonky3/Plonky3). This sibling workspace contains four
new crates; existing Plonky3 crates are fetched as Git dependencies.

There are two recursion paths:

- **Prime-field circuits:** `p3-circuit` builds a fixed table machine, proves its
  operations with batch STARKs, and connects tables through shared wire lookups.
- **Binary-field Flock:** the upstream `p3-flock` dependency proves batched Boolean
  R1CS instances over binary fields and builds recursive verifier circuits.
  XMSS and SLH-DSA aggregation use this path.

The APIs and proof formats are experimental and may change. Security properties
and assumptions depend on the selected protocol and configuration; consult the
crate documentation and source before integrating them.

## Crates

| Crate | Purpose |
| --- | --- |
| [`p3-verifier-arith`](verifier-arith) | Shared arithmetic interface for native and circuit verifier execution. |
| [`p3-circuit`](circuit) | Prime-field circuit construction, proving, verification, and recursive aggregation. |
| [`p3-xmss`](xmss) | XMSS-style signatures with keyed BLAKE3 and binary Flock aggregation and recursion. |
| [`p3-sphincs`](sphincs) | SLH-DSA-SHA2-128s verification and aggregation through binary Flock proofs. |

## Upstream dependencies

All existing Plonky3 crates are pinned to immutable
[`recursion-lab` commit `9d4e1ce0c4a5875072b354800e046ce0c17ce6e2`](https://github.com/Plonky3/Plonky3/commit/9d4e1ce0c4a5875072b354800e046ce0c17ce6e2).
The required generic verifier and binary recursion APIs are present on that
branch and are absent from
[the upstream main revision assessed for this extraction, `716e00b3b02da9e0cc3b3c78f2441cb4ca1eddca`](https://github.com/Plonky3/Plonky3/commit/716e00b3b02da9e0cc3b3c78f2441cb4ca1eddca).
Switching directly to that main revision requires porting those APIs first.
No existing Plonky3 crate is vendored in this repository.

The single Git patch in [`Cargo.toml`](Cargo.toml) routes upstream users of
`p3-verifier-arith` to this workspace's local crate, so native verifiers and
circuits share the same arithmetic traits. Cargo only honors patches in the
consuming workspace root. Downstream projects using these crates must add the
corresponding `p3-verifier-arith` patch to their own root manifest, using a path
or Git reference appropriate to that project.

For a local sibling checkout:

```toml
[patch."https://github.com/Plonky3/Plonky3"]
p3-verifier-arith = { path = "../Plonky3-native-recursion/verifier-arith" }
```

Git consumers should use the same sibling repository revision as their other
recursion crate dependencies.

## Build and check

Use stable Rust for builds and Clippy, and nightly rustfmt for formatting:

```bash
rustup component add clippy
rustup toolchain install nightly --component rustfmt
cargo test --workspace
cargo test --workspace --all-features
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo +nightly fmt --all -- --check
```

For faster test execution, CI sets `CARGO_PROFILE_TEST_OPT_LEVEL=1`. The signature
crates expose an optional `parallel` feature.
`RUSTFLAGS="-Ctarget-cpu=native"` enables optimizations supported by the host CPU.

## Try the recursion paths

Start with the prime-field table tests and the circuit recursion smoke tests:

```bash
cargo test -p p3-circuit --test tables
cargo test -p p3-circuit --test recursion_smoke
```

Signature aggregation examples are larger workloads:

```bash
RUSTFLAGS="-Ctarget-cpu=native" cargo run --release -p p3-xmss --features parallel --example aggregate_binary
RUSTFLAGS="-Ctarget-cpu=native" cargo run --release -p p3-sphincs --features parallel --example aggregate -- 1
```

Both signature crates also provide an `aggregate_tree` example; select the
package explicitly because the example name is shared.

## Heavy recursion tests

Ordinary workspace tests skip ignored tests. Large recursion tests should be run
individually in release mode on a machine with sufficient memory. The source
annotations estimate peak memory under 20 GB for XMSS recursion and about 27 GB
for SLH-DSA aggregation. Actual requirements depend on the configuration and
machine. These workloads are not part of the routine CI test matrix.

```bash
cargo test --release -p p3-xmss --test recursion -- --ignored --nocapture
cargo test --release -p p3-sphincs --test aggregate -- --ignored --nocapture
```

Prime-field recursive circuit aggregation also has ignored tests in
[`circuit/tests/aggregate.rs`](circuit/tests/aggregate.rs). Run those individually:

```bash
cargo test --release -p p3-circuit --test aggregate four_proofs_aggregate_into_one_root -- --ignored --nocapture
cargo test --release -p p3-circuit --test aggregate public_keys_come_before_the_digest -- --ignored --nocapture
```

## Source and attribution

The four crates were extracted from Plonky3's
[`recursion-lab` branch](https://github.com/Plonky3/Plonky3/tree/recursion-lab) at
[source commit `9d4e1ce0c4a5875072b354800e046ce0c17ce6e2`](https://github.com/Plonky3/Plonky3/commit/9d4e1ce0c4a5875072b354800e046ce0c17ce6e2).
The imported history preserves original authors, commit messages, and
attribution trailers for these paths. Filtering the history changes commit
hashes. Standalone workspace configuration and repository documentation are
maintained here.

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidance.

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or
  <https://www.apache.org/licenses/LICENSE-2.0>)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or
  <https://opensource.org/licenses/MIT>)

at your option.

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
