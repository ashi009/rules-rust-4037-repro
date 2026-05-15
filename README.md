# Minimal reproducer for [bazelbuild/rules_rust#4037](https://github.com/bazelbuild/rules_rust/issues/4037)

With rules_rust 0.70.0 and `generate_cargo_toml_env_vars = True` (the default
in `crate.from_cargo(...)`), a 3rd-party crate's `build.rs` does not see
`CARGO_PKG_*` at execution time. The compile-time env file is wired
(`rustc_env_files = [":cargo_toml_env_vars"]` on the `cargo_build_script`),
but the runtime env file (`build_script_env_files`) is silently dropped
because the `CargoBuildScript` starlark struct in
`crate_universe/src/utils/starlark.rs` has no `build_script_env_files` field.

The `rav1e` crate uses [`built`](https://crates.io/crates/built) in its
`build.rs`, which reads `env::var("CARGO_PKG_AUTHORS")` at execution time, so
it's a convenient real-world tripwire.

## Reproduce

```
bazel build @crates//:rav1e
```

Expected (and observed against unpatched rules_rust 0.70.0):

```
thread 'main' panicked at external/rules_rust++crate+crates__built-0.7.7/src/environment.rs:54:9:
Missing expected environment variable "CARGO_PKG_AUTHORS"
...
Target @@rules_rust++crate+crates__rav1e-0.7.1//:rav1e failed to build
```

## Versions

- `rules_rust`: 0.70.0
- Bazel: tested with 7.7.1
- `rav1e`: 0.7.1 (pulled from crates.io via `crate.from_cargo`)
