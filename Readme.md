---
title: "Macro Madness"
author: Pascal Hertleif
date: 2019-07-09
categories:
- rust
- presentation
progress: true
slideNumber: true
history: true
navigation: false
controls: 0
---

# Hi, I'm Pascal Hertleif

- Senior dev here at TTC
- Dev tools team
- CLI WG
- {[twitter],[github]}.com/killercup

[twitter]: https://twitter.com/killercup
[github]: https://github.com/killercup

# What this talk will teach you

- - -

![ ](images/before.svg)

- - -

![ ](images/after.svg)

- - -

Just kidding!

This topic is _way_ too large for this short a talk

- - -

I just don't want to write the same code over and over again

. . .

I also like to make up abstract solution

. . .

This is a dangerous combination…


# Case Study 1: A repetitive `impl`

- - -

A derive macro for a specialized `Debug`

Code: [github.com/killercup/snafu-cli-debug](https://github.com/killercup/snafu-cli-debug)

# Case Study 2: A byte pattern

- - -

Task: Find a pattern like `fd fd ? ? df 1e` in a slice of bytes

Idea: Use regex!

- - -

But how do turn that pattern format into a valid regular expression?

With a macro of course! And so we dove into the world of `macro_rules!` macros…

## Token trees

Problem: Can't parse a string in pattern macros

. . .

Approach: Munch single tokens one after the other and concat to a string!

```rust
byte_pattern!(fd fd ? ? df 1e)
```

. . .

cf. [The Little Book of Rust Macros](https://danielkeep.github.io/tlborm/book/pat-incremental-tt-munchers.html)

## Limits of `macro_rules!`

Except that macros operate on Rust tokens…

```console
error: expected at least one digit in exponent
 --> src/main.rs:6:34
  |
6 |     byte_pattern!(fd fd ? ? df 1e)
  |                                  ^
```

([code on playpen](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=e6b702cf4d088c73676cc0bf014359fe))

## And a proc it was

Since function-like macros aren't stable yet, we went for a `derive`

```rust
#[derive(Debug, BytePattern)]
#[byte_pattern="fd fd (? ?) df 1e"]
struct MyMatch<'source> {
  capture: &'source [u8; 2],
}
```

## In case of errors, add more crates

Crates that are `proc-macro = true` are a bit special

For example: [Adding nom as a dependency to a proc-macro crate with tests results in link error](https://github.com/rust-lang/rust/issues/62146)

::: notes

Splitting implementations up into multiple crates makes most errors go away,
except for what to call them.

Ideas for names:

- `-core`
- `-common`
- `-shared`
- `-lib`
- `-derive`
- `-macros`
- `-base`
- `-helpers`
- `-utils`

:::

# Case Study 3: This escalated quickly

- - -

> 'twas a fortnight ago, when I found myself in the peculiar situation of having
> written a rather complex PEG grammar, that I then had to parse into an AST
> of sorts…

::: notes

Ramble on and on about the joy of parsing stuff and building ASTs.

…for however much time is still available.

:::

- - -

Long story short:

- [pest](https://pest.rs/) is a parser using its own grammer language
- [pest-ast](https://github.com/pest-parser/ast) is a derive macro for building
data types from pest token trees
- [pest-ast-generator](https://github.com/killercup/pest-ast-generator) is a
  derive macro that generates the structs that it calls `pest-ast` on by
  reading the original grammar

# Thank you!

I'm Pascal and I want you to talk to me!

{[twitter],[github]}.com/killercup

Slides: [git.io/rust-macro-madness](https://git.io/rust-macro-madness)
