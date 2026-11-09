# rules_rs repro: `dep/feature` on an optional dependency enables a feature that doesn't exist

env_logger 0.11 declares

```toml
[features]
color = ["dep:anstream", "dep:anstyle"]
auto-color = ["color", "anstream/auto"]
```

When `auto-color` is enabled, `anstream/auto` should enable the optional `anstream` dependency with `auto` enabled:

```
$ cargo tree -e normal -f '{p}|{f}' --prefix none | grep ^env_logger
env_logger v0.11.11|auto-color,color

$ cargo build -v 2>&1 | grep -o 'crate-name env_logger.*' | grep -o 'feature="[^"]*"'
feature="auto-color"
feature="color"
```

rules_rs 0.2.1 incorrectly enables a feature `anstream` on env_logger:

```
$ bazel build //:repro
$ grep crate_features "$(bazel info output_base)"/external/rules_rs++crate+crates__env_logger-0.11.11/BUILD.bazel
    crate_features = ["anstream", "auto-color", "color"],
```

so rustc gets a `--cfg feature="anstream"` for a feature that doesn't exist
