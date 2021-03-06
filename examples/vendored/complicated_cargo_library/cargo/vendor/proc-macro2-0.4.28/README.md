# proc-macro2

[![Build Status](https://api.travis-ci.com/alexcrichton/proc-macro2.svg?branch=master)](https://travis-ci.com/alexcrichton/proc-macro2)
[![Latest Version](https://img.shields.io/crates/v/proc-macro2.svg)](https://crates.io/crates/proc-macro2)
[![Rust Documentation](https://img.shields.io/badge/api-rustdoc-blue.svg)](https://docs.rs/proc-macro2)

A wrapper around the procedural macro API of the compiler's `proc_macro` crate.
This library serves three purposes:

- **Bring proc-macro-like functionality to other contexts like build.rs and
  main.rs.** Types from `proc_macro` are entirely specific to procedural macros
  and cannot ever exist in code outside of a procedural macro. Meanwhile
  `proc_macro2` types may exist anywhere including non-macro code. By developing
  foundational libraries like [syn] and [quote] against `proc_macro2` rather
  than `proc_macro`, the procedural macro ecosystem becomes easily applicable to
  many other use cases and we avoid reimplementing non-macro equivalents of
  those libraries.

- **Make procedural macros unit testable.** As a consequence of being specific
  to procedural macros, nothing that uses `proc_macro` can be executed from a
  unit test. In order for helper libraries or components of a macro to be
  testable in isolation, they must be implemented using `proc_macro2`.

- **Provide the latest and greatest APIs across all compiler versions.**
  Procedural macros were first introduced to Rust in 1.15.0 with an extremely
  minimal interface. Since then, many improvements have landed to make macros
  more flexible and easier to write. This library tracks the procedural macro
  API of the most recent stable compiler but employs a polyfill to provide that
  API consistently across any compiler since 1.15.0.

[syn]: https://github.com/dtolnay/syn
[quote]: https://github.com/dtolnay/quote

## Usage

```toml
[dependencies]
proc-macro2 = "0.4"
```

The skeleton of a typical procedural macro typically looks like this:

```rust
extern crate proc_macro;

#[proc_macro_derive(MyDerive)]
pub fn my_derive(input: proc_macro::TokenStream) -> proc_macro::TokenStream {
    let input = proc_macro2::TokenStream::from(input);

    let output: proc_macro2::TokenStream = {
        /* transform input */
    };

    proc_macro::TokenStream::from(output)
}
```

If parsing with [Syn], you'll use [`parse_macro_input!`] instead to propagate
parse errors correctly back to the compiler when parsing fails.

[`parse_macro_input!`]: https://docs.rs/syn/0.15/syn/macro.parse_macro_input.html

## Unstable features

The default feature set of proc-macro2 tracks the most recent stable compiler
API. Functionality in `proc_macro` that is not yet stable is not exposed by
proc-macro2 by default.

To opt into the additional APIs available in the most recent nightly compiler,
the `procmacro2_semver_exempt` config flag must be passed to rustc. As usual, we
will polyfill those nightly-only APIs all the way back to Rust 1.15.0. As these
are unstable APIs that track the nightly compiler, minor versions of proc-macro2
may make breaking changes to them at any time.

```
RUSTFLAGS='--cfg procmacro2_semver_exempt' cargo build
```

Note that this must not only be done for your crate, but for any crate that
depends on your crate. This infectious nature is intentional, as it serves as a
reminder that you are outside of the normal semver guarantees.

Semver exempt methods are marked as such in the proc-macro2 documentation.

# License

This project is licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
   http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or
   http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in Serde by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
