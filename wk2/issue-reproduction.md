# Issue Reproduction Log

## Original Run (no changes to code):

Shows a successful `make cargo-clippy` run (expected behavior, as `#[allow]` hasn't been removed.

OUTPUT:

```
  Compiling zerofrom-derive v0.1.4
   Compiling yoke-derive v0.7.4
    Checking winapi-util v0.1.6
    Checking clap_builder v4.5.60
    Checking futures-util v0.3.32
   Compiling clap_derive v4.5.55
   Compiling ref-cast-impl v1.0.24
   Compiling onig_sys v69.9.1
   Compiling prost-derive v0.13.5
    Checking pin-project v1.1.11
    Checking tracing v0.1.44
    Checking zstd v0.13.2
    Checking thiserror v2.0.17
    Checking ref-cast v1.0.24
    Checking supports-color v2.1.0
    Checking dirs-sys-next v0.1.2
   Compiling thiserror-impl v1.0.68
   Compiling prost v0.12.6
   Compiling strum_macros v0.28.0
   Compiling snafu-derive v0.7.5
    Checking owo-colors v4.2.3
    Checking dirs-next v2.0.0
    Checking env_logger v0.11.9
   Compiling num_enum_derive v0.6.1
    Checking zerofrom v0.1.4
    Checking aes v0.8.3
   Compiling prost-types v0.12.6
    Checking yoke v0.7.4
   ...
```

## New Run (allow statements have been removed):

Shows a successful `make cargo-clippy` run (expected behavior, as the removed statements were stale and no longer necessary).

OUTPUT:

```
 Blocking waiting for file lock on build directory
    Checking serde v1.0.228
   Compiling ring v0.17.14
    Checking zerovec v0.10.4
    Checking futures-executor v0.3.32
   Compiling darling_macro v0.20.11
   Compiling prost-build v0.12.6
   Compiling onig_sys v69.9.1
    Checking clap v4.5.60
    Checking thiserror v1.0.68
    Checking strum v0.28.0
   Compiling darling_core v0.23.0
   Compiling snafu-derive v0.9.0
    Checking half v2.4.1
    Checking regex-filtered v0.2.0
   Compiling chrono-tz v0.10.4
    Checking cmac v0.7.2
   Compiling pest_generator v2.8.6
    Checking futures v0.3.32
    Checking snafu v0.7.5
    Checking num_enum v0.6.1
    Checking term v0.7.0
    Checking termcolor v1.3.0
    Checking retry-policies v0.5.1
    Checking salsa20 v0.10.2
    Checking ciborium-ll v0.2.2
    Checking chacha20 v0.9.1
    Checking ctr v0.9.2
    Checking sha2 v0.10.9
    Checking smallvec v1.15.1
    Checking bytes v1.11.1
    Checking ipnet v2.12.0
    Checking num-bigint v0.4.6
    Checking serde_urlencoded v0.7.1
    Checking chrono v0.4.44
    Checking ahash v0.8.11
    Checking encoding_rs v0.8.35
    Checking parking_lot_core v0.9.12
   Compiling ua-parser v0.2.0
   Compiling serde_yaml_ng v0.10.0
    Checking parking_lot v0.12.5
    Checking fluent-uri v0.4.1
    Checking http v0.2.12
    Checking http v1.3.1
    Checking prost v0.13.5
    Checking tokio v1.52.2
    Checking tinystr v0.7.6
    Checking icu_collections v1.5.0
    Checking octseq v0.6.1
    Checking prost-types v0.13.5
   ...
```

## New Run
