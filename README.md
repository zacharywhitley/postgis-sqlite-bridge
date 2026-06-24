# postgis-sqlite-bridge

Generated SQLite loadable extension that bridges the **postgis** DataFission wasm shim into SQLite as native scalar functions, aggregates, and UDTFs via rusqlite's `create_scalar_function` / `create_aggregate_function` / `VTab` traits.

Produced by [`sqlink-shim-codegen`](https://github.com/zacharywhitley/sqlink-shim-codegen) from a shim-interface SQLite database. **Do not edit by hand** — regenerate from the source.

## Surface

| Extension | Version | Scalars | Aggregates | UDTFs | Windows | Types | Operators | Casts | Preprocessors | Catalog | Indexes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| postgis | 0.1.0 | 402 | 11 | 7 | 4 | 7 | 23 | 4 | 1 | 0 | 9 |

## Build

```sh
cargo build --release
```

The build needs sibling checkouts of the path-dep'd workspace
crates (`datafission-df-plugin-loader`, `datafission-df-plugin-api`,
`datafission-functions`) at `../datafission/crates/`.

## Load + use

The bridge needs the composed shim wasm at runtime; set
`POSTGIS_SHIM_WASM` before `.load`:

```sh
POSTGIS_SHIM_WASM=/path/to/postgis-composed.wasm \
  sqlite3 -cmd ".load target/release/libpostgis_sqlite_bridge.dylib" :memory:
```

**macOS gotcha**: the system `/usr/bin/sqlite3` is compiled
with `-DSQLITE_OMIT_LOAD_EXTENSION` and refuses `.load`. Use
homebrew sqlite (`/opt/homebrew/Cellar/sqlite/<ver>/bin/sqlite3`)
or any distro sqlite that ships extension support.

## Regen

When the upstream shim's SQL surface changes:

```sh
cd ~/git/sqlink-shim-codegen
cargo run --release -- \
  --interface /path/to/postgis-interface.sqlite \
  --out ~/git/postgis-sqlite-bridge
```

The codegen pipes every emitted `.rs` through
`rustfmt --edition 2021`, so the resulting crate is
`cargo fmt -p postgis-sqlite-bridge -- --check`-clean by
construction.

## Architecture

- Scalars: registered via rusqlite's
  `Connection::create_scalar_function_with_state`, dispatched
  one row at a time (SQLite is not vectorised).
- Aggregates: rusqlite's `Aggregate` trait with state held
  per-group in `Box<dyn Accumulator>`.
- UDTFs: rusqlite VTab eponymous tables; output schema inferred
  from `def.param_types()`.

## License

Apache-2.0. Generated source so the same license as the
codegen.
