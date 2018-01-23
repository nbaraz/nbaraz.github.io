# Exhaustive asm

Inline asm poses a bit of a problem - code containing it will only compile for a single
architecture, unless cfg attributes are used. I propose to:

* Force crates to specify supported architectures in their Cargo.toml (default is 'all'). The list
  is passed to the compiler.
* Asm statements are given the architecture as a parameter (or perhaps have an `asm_<arch>` macro
  per architecture).
* Asm statements can be placed in 2 locations:
    * An `arch_match!` procedural macro
    * A function marked with a special per-architecture attribute (`#[architecture(<arch>)]`)
        * These functions can call eachother, like unsafe functions can

#### arch-match
A macro that opens an architecture specific scope, per architecture. must be exhaustive across all
crate-supported architectures.

Example usage:
```rust
arch_match! {
    x86 => asm_x86!(...),
    armv7 => asm_armv7(...),
    all => unoptimized_rust_function(),
}
```

If for some reason, some asm statement is only relevant for a single architecture, one can just
match on it and 'all', and do nothing in 'all':
```rust
arch_match! {
    x86 => asm_x86!("important_instruction"),
    all => { },
}
```

#### Asm subtyping
I don't know exactly how this should work, but for example, newer amd64 cpus subtype older amd64
cpus. Everything subtypes 'all'. Maybe it should be more trait-like? (specify `amd64 + sse2` as the architecture)

#### The 'all' architecture
Only rust code can run in this context.
