# Textwrap

[![](https://travis-ci.org/mgeisler/textwrap.svg?branch=master)][travis-ci]
[![](https://ci.appveyor.com/api/projects/status/github/mgeisler/textwrap?branch=master&svg=true)][appveyor]
[![](https://codecov.io/gh/mgeisler/textwrap/branch/master/graph/badge.svg)][codecov]
[![](https://img.shields.io/crates/v/textwrap.svg)][crates-io]
[![](https://docs.rs/textwrap/badge.svg)][api-docs]

Textwrap is a small Rust crate for word wrapping text. You can use it
to format strings for display in commandline applications. The crate
name and interface is inspired by
the [Python textwrap module][py-textwrap].

## Usage

To use `textwrap`, add this to your `Cargo.toml` file:
```toml
[dependencies]
textwrap = "0.11"
```

This gives you the text wrapping without of the optional features
listed next.

### `hyphenation`

If you would like to have automatic language-sensitive hyphenation,
enable the `hyphenation` feature:

```toml
[dependencies]
textwrap = { version = "0.11", features = ["hyphenation"] }
```

This gives you hyphenation support for US English. Please see the
[`hyphenation` example] for an executable demo. Read the Getting
Started section below to see how to load the hyphenation patterns for
other languages.

### `terminal_size`

To conveniently wrap text at the current terminal width, enable the
`terminal_size` feature:

```toml
[dependencies]
textwrap = { version = "0.11", features = ["terminal_size"] }
```

Please see the [`termwidth` example] for how to use this feature.

## Documentation

**[API documentation][api-docs]**

## Getting Started

Word wrapping single strings is easy using the `fill` function:
```rust
fn main() {
    let text = "textwrap: a small library for wrapping text.";
    println!("{}", textwrap::fill(text, 18));
}
```
The output is
```
textwrap: a small
library for
wrapping text.
```

If you enable the `hyphenation` feature, you get support for automatic
hyphenation for [about 70 languages][patterns] via high-quality TeX
hyphenation patterns.

Your program must load the hyphenation pattern and call
`Wrapper::with_splitter` to use it:

```rust
use hyphenation::{Language, Load, Standard};
use textwrap::Wrapper;

fn main() {
    let hyphenator = Standard::from_embedded(Language::EnglishUS).unwrap();
    let wrapper = Wrapper::with_splitter(18, hyphenator);
    let text = "textwrap: a small library for wrapping text.";
    println!("{}", wrapper.fill(text))
}
```

The output now looks like this:
```
textwrap: a small
library for wrap-
ping text.
```

The US-English hyphenation patterns are embedded when you enable the
`hyphenation` feature. They are licensed under a [permissive
license][en-us license] and take up about 88 KB of space in your
application. If you need hyphenation for other languages, you need to
download a [precompiled `.bincode` file][bincode] and load it
yourself. Please see the [`hyphenation` documentation] for details.

## Examples

The library comes with some small example programs that shows various
features.

### Layout Example

The `layout` example shows how a fixed example string is wrapped at
different widths. Run the example with:

```shell
$ cargo run --features hyphenation --example layout
```

The program will use the following string:

> Memory safety without garbage collection. Concurrency without data
> races. Zero-cost abstractions.

The string is wrapped at all widths between 15 and 60 columns. With
narrow columns the output looks like this:

```
.--- Width: 15 ---.
| Memory safety   |
| without garbage |
| collection.     |
| Concurrency     |
| without data    |
| races. Zero-    |
| cost abstrac-   |
| tions.          |
.--- Width: 16 ----.
| Memory safety    |
| without garbage  |
| collection. Con- |
| currency without |
| data races. Ze-  |
| ro-cost abstrac- |
| tions.           |
```

Later, longer lines are used and the output now looks like this:

```
.-------------------- Width: 49 --------------------.
| Memory safety without garbage collection. Concur- |
| rency without data races. Zero-cost abstractions. |
.---------------------- Width: 53 ----------------------.
| Memory safety without garbage collection. Concurrency |
| without data races. Zero-cost abstractions.           |
.------------------------- Width: 59 -------------------------.
| Memory safety without garbage collection. Concurrency with- |
| out data races. Zero-cost abstractions.                     |
```

Notice how words are split at hyphens (such as "zero-cost") but also
how words are hyphenated using automatic/machine hyphenation.

### Terminal Width Example

The `termwidth` example simply shows how the width can be set
automatically to the current terminal width. Run it with this command:

```
$ cargo run --example termwidth
```

If you run it in a narrow terminal, you'll see output like this:
```
Formatted in within 60 columns:
----
Memory safety without garbage collection. Concurrency
without data races. Zero-cost abstractions.
----
```

If `stdout` is not connected to the terminal, the program will use a
default of 80 columns for the width:

```
$ cargo run --example termwidth | cat
Formatted in within 80 columns:
----
Memory safety without garbage collection. Concurrency without data races. Zero-
cost abstractions.
----
```

## Release History

This section lists the largest changes per release.

### Unreleased ###

We now require the [Rust 2018 edition][rust-2018]. Over the years,
we’ve repeatedly seen build failures in our CI, even when nothing
changed in `textwrap`. The failures happened because we tested against
a fixed version of Rust, but our dependencies kept releasing new patch
it has versions that would push up the minimum required Rust version.

The build failures makes it infeasible to keep `textwrap` compatible
with any particular version of Rust. We will therefore track the
latest stable version of Rust from now on.

The `term_size` feature has been replaced by `terminal_size`. The API
is unchanged, it is just the name of the Cargo feature that changed.

The `hyphenation` feature now only embeds the hyphenation patterns for
US-English. This slims down the dependency.

* Fixed [#177][issue-177]: Update examples to the 2018 edition.

### Version 0.11.0 — December 9th, 2018

Due to our dependencies bumping their minimum supported version of
Rust, the minimum version of Rust we test against is now 1.22.0.

* Merged [#141][issue-141]: Fix `dedent` handling of empty lines and
  trailing newlines. Thanks @bbqsrc!
* Fixed [#151][issue-151]: Release of version with hyphenation 0.7.

### Version 0.10.0 — April 28th, 2018

Due to our dependencies bumping their minimum supported version of
Rust, the minimum version of Rust we test against is now 1.17.0.

* Fixed [#99][issue-99]: Word broken even though it would fit on line.
* Fixed [#107][issue-107]: Automatic hyphenation is off by one.
* Fixed [#122][issue-122]: Take newlines into account when wrapping.
* Fixed [#129][issue-129]: Panic on string with em-dash.

### Version 0.9.0 — October 5th, 2017

The dependency on `term_size` is now optional, and by default this
feature is not enabled. This is a *breaking change* for users of
`Wrapper::with_termwidth`. Enable the `term_size` feature to restore
the old functionality.

Added a regression test for the case where `width` is set to
`usize::MAX`, thanks @Fraser999! All public structs now implement
`Debug`, thanks @hcpl!

* Fixed [#101][issue-101]: Make `term_size` an optional dependency.

### Version 0.8.0 — September 4th, 2017

The `Wrapper` stuct is now generic over the type of word splitter
being used. This means less boxing and a nicer API. The
`Wrapper::word_splitter` method has been removed. This is a *breaking
API change* if you used the method to change the word splitter.

The `Wrapper` struct has two new methods that will wrap the input text
lazily: `Wrapper::wrap_iter` and `Wrapper::into_wrap_iter`. Use those
if you will be iterating over the wrapped lines one by one.

* Fixed [#59][issue-59]: `wrap` could return an iterator. Thanks
  @hcpl!
* Fixed [#81][issue-81]: Set `html_root_url`.

### Version 0.7.0 — July 20th, 2017

Version 0.7.0 changes the return type of `Wrapper::wrap` from
`Vec<String>` to `Vec<Cow<'a, str>>`. This means that the output lines
borrow data from the input string. This is a *breaking API change* if
you relied on the exact return type of `Wrapper::wrap`. Callers of the
`textwrap::fill` convenience function will see no breakage.

The above change and other optimizations makes version 0.7.0 roughly
15-30% faster than version 0.6.0.

The `squeeze_whitespace` option has been removed since it was
complicating the above optimization. Let us know if this option is
important for you so we can provide a work around.

* Fixed [#58][issue-58]: Add a "fast_wrap" function.
* Fixed [#61][issue-61]: Documentation errors.

### Version 0.6.0 — May 22nd, 2017

Version 0.6.0 adds builder methods to `Wrapper` for easy one-line
initialization and configuration:

```rust
let wrapper = Wrapper::new(60).break_words(false);
```

It also add a new `NoHyphenation` word splitter that will never split
words, not even at existing hyphens.

* Fixed [#28][issue-28]: Support not squeezing whitespace.

### Version 0.5.0 — May 15th, 2017

Version 0.5.0 has *breaking API changes*. However, this only affects
code using the hyphenation feature. The feature is now optional, so
you will first need to enable the `hyphenation` feature as described
above. Afterwards, please change your code from
```rust
wrapper.corpus = Some(&corpus);
```
to
```rust
wrapper.splitter = Box::new(corpus);
```

Other changes include optimizations, so version 0.5.0 is roughly
10-15% faster than version 0.4.0.

* Fixed [#19][issue-19]: Add support for finding terminal size.
* Fixed [#25][issue-25]: Handle words longer than `self.width`.
* Fixed [#26][issue-26]: Support custom indentation.
* Fixed [#36][issue-36]: Support building without `hyphenation`.
* Fixed [#39][issue-39]: Respect non-breaking spaces.

### Version 0.4.0 — January 24th, 2017

Documented complexities and tested these via `cargo bench`.

* Fixed [#13][issue-13]: Immediatedly add word if it fits.
* Fixed [#14][issue-14]: Avoid splitting on initial hyphens.

### Version 0.3.0 — January 7th, 2017

Added support for automatic hyphenation.

### Version 0.2.0 — December 28th, 2016

Introduced `Wrapper` struct. Added support for wrapping on hyphens.

### Version 0.1.0 — December 17th, 2016

First public release with support for wrapping strings on whitespace.

## License

Textwrap can be distributed according to the [MIT license][mit].
Contributions will be accepted under the same license.

[crates-io]: https://crates.io/crates/textwrap
[travis-ci]: https://travis-ci.org/mgeisler/textwrap
[appveyor]: https://ci.appveyor.com/project/mgeisler/textwrap
[codecov]: https://codecov.io/gh/mgeisler/textwrap
[py-textwrap]: https://docs.python.org/library/textwrap
[`hyphenation` example]: https://github.com/mgeisler/textwrap/blob/master/examples/hyphenation.rs
[`termwidth` example]: https://github.com/mgeisler/textwrap/blob/master/examples/termwidth.rs
[patterns]: https://github.com/tapeinosyne/hyphenation/tree/master/patterns-tex
[en-us license]: https://github.com/hyphenation/tex-hyphen/blob/master/hyph-utf8/tex/generic/hyph-utf8/patterns/tex/hyph-en-us.tex
[bincode]: https://github.com/tapeinosyne/hyphenation/tree/master/dictionaries
[`hyphenation` documentation]: http://docs.rs/hyphenation
[api-docs]: https://docs.rs/textwrap/
[rust-2018]: https://doc.rust-lang.org/edition-guide/rust-2018/
[issue-13]: https://github.com/mgeisler/textwrap/issues/13
[issue-14]: https://github.com/mgeisler/textwrap/issues/14
[issue-19]: https://github.com/mgeisler/textwrap/issues/19
[issue-25]: https://github.com/mgeisler/textwrap/issues/25
[issue-26]: https://github.com/mgeisler/textwrap/issues/26
[issue-28]: https://github.com/mgeisler/textwrap/issues/28
[issue-36]: https://github.com/mgeisler/textwrap/issues/36
[issue-39]: https://github.com/mgeisler/textwrap/issues/39
[issue-58]: https://github.com/mgeisler/textwrap/issues/58
[issue-59]: https://github.com/mgeisler/textwrap/issues/59
[issue-61]: https://github.com/mgeisler/textwrap/issues/61
[issue-81]: https://github.com/mgeisler/textwrap/issues/81
[issue-99]: https://github.com/mgeisler/textwrap/issues/99
[issue-101]: https://github.com/mgeisler/textwrap/issues/101
[issue-107]: https://github.com/mgeisler/textwrap/issues/107
[issue-122]: https://github.com/mgeisler/textwrap/issues/122
[issue-129]: https://github.com/mgeisler/textwrap/issues/129
[issue-141]: https://github.com/mgeisler/textwrap/issues/141
[issue-151]: https://github.com/mgeisler/textwrap/issues/151
[issue-177]: https://github.com/mgeisler/textwrap/issues/177
[mit]: LICENSE
