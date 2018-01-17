# Immovable types and self-referencing structs

[Original RFC by Zoxc](https://github.com/rust-lang/rfcs/pull/1858)

I'm writing this because discussion has stagnated a bit, and I think that some things are missing from the RFC (namely interaction with placement-new). The only thing from the that I really disgree with from the original RFC is `MovableCell`, which as far as I can tell cannot be made safe.

## Use cases

As far as I can see, there are 3 use cases for immovable types:
* Self-referencing structs (and more specifically, generators!). Can never be moved after construction.
* Structs whose address is observed by FFI functions (Less strict than self-referencing structs)
    * There is no problem with moving this type of struct until a reference is taken (explained in the original rfc). Self referencing structs are a subset of this requirement - they are simply referenced on creation.
    * I might neglect this use in this document, because it has the least strict requirements.
* Unleakable structs (explanation in its own section)

## Feature interaction - placement-new
There is heavy interaction between these two features. Everything return value and function argument in rust is moved (this is more relevant to return values). If placement-new moves the value behind the scenes, it needs to be changed.

Immovable types can be un-returnable as a start - just place-able. This means that in order to let an immovable type escape a function, it needs to take a `Place` parameter.  (See [Place on return][#Place on return]).

## Details
* Add a new auto-trait* `Move`, which is implemented for every struct, except self-referencing ones. It can also be opted out of.
* A `Move` trait bound is always implied.
* References and pointers are always `Move`.

(A little lacking)

### Destruction (self-referencing structs)
Haven't thought about this much. Drop order matters.
Might need an intrinsic in order to allow early-dropping immovable structs (`mem::drop` moves the value).

## Ergonomics
Immovable structs can be implemented with bad ergonomics - I think if is better to focus on soundness and worry about ergonomics later. However, there is no reason for me to keep my thoughts about this to myself.

### Construction
Currently, there is no way to construct a self-referencing struct.
This isn't as important as it may seem, since the construction **can** be done unsafely - once the struct is complete, it is safe to use.

TODO: Syntax proposal.

### Place on return
Being unable to return immovable structs isn't very ergonomic. This can happen behind the scenes - functions returning immovable types can be transformed to receive a `Place` parameter. Call sites must either be a local variable binding ([Stack place][#Stack place]), or a `placement-in` expression (or is it a statement?).

## Supporting features
These are features that could work well in combination with immovable types.

### Stack place
A `Place` that is implicitly created and passed to functions which return immovable types. I'm not sure if this is a good idea, or how hard it is to implement.

### ExplicitMove trait
Sometimes you might want to move an immovable struct after it has been constructed.

Functionally like CPP's move constructor.
It is implemented for every struct whose members implement `ExplicitMove`. Unlike CPP, never called implicitly.

A little dangerous to add, since people might add this bound everywhere. If rust could do without immovable structs, it can do without immovable structs that can be moved after construction.

## Unleakable structs
We can use this feature to create types which cannot be leaked by safe code (as far as I can tell). If we add a `StackOnly: !Move` trait, which marks the struct as only place-able in a `StackPlace`, they can not be moved from the stack, and their destructor can only be called at/before scope exit (before == with drop intrinsic).

## Appendix
I don't know how to format this section properly in markdown, this will do for now.

```
* I don't know the correct term for this type of thing - I would be glad if someone referred me to an explanation/explained it to me.
```
