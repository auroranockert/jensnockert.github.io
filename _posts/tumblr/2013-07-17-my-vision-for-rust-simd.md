---
layout: post
title: My Vision for Rust SIMD
date: '2013-07-17T07:34:00+02:00'
tags:
- Rust
- SIMD
tumblr_url: http://blog.aventine.se/post/55669497784/my-vision-for-rust-simd
---

It is no secret that I have on the agenda to try and make Rust into an awesome language for numerics, DSP and similar applications, where low-level programming can be required for speed. Rust is a perfect fit since it is memory-safe, yet doesn't have a GC and generally makes few design decisions that make it significantly slower than C, and to follow up that tradition with a design that improves on the performance of C, I have implemented OpenCL-style vectors in Rust [on Github](https://github.com/jensnockert/rust/tree/simd).

The patch isn't by any means finished, and I won't finish it until I think I have a good chance to have it merged. The first 90% are done, so I would just have 90% left. And before I commit to those other 90%, I would like to  facilitate some discussion, and make sure that what we get in relation to SIMD in the end is awesome.

This may be bikeshedding, but I really enjoy bikeshedding, and I hope that we can improve the design by doing it.


Syntax for Types
--------------------------------------------------------------------------------

My preference for declaring a type would be that it would be fully anonymous, and you simply cannot give them names with syntax mirroring fixed-length vectors, `<T, ..n>`, for example `<f32, ..4>` for the most common single-precision format.

If that does not fit with the grammar, in order of preference, the following would be reasonable alternatives.

 1. `simd!(f32 * 4)`
 2. `#[simd(4)] f32`
 3. `#[simd] [f32, ..4]`
 4. `#[simd] (f32, f32, f32, f32)`

Forcing names on SIMD types (like the current syntax) is not a good idea, for various reasons, most of them involving comparison, shuffles, intrinsics and so on, is a really bad idea.

Consider the SSE2 intrinsic `_mm_mul_epu32` for example, which takes two `<u32, ..4>` and returns a `<u64, ..2>`, if we enforce that all SIMD types should have a name, what should the name of the return type of this intrinsic be?

Another example, if we allow `a == b` to mean componentwise comparison of a vector, what would the resulting type be? If we don't name types, we can just say that it is a `<bool, ..n>`, but if we do name types, which of the names should we pick?

Last, but possibly most convincing example, since we cannot possibly pre-declare all of the types. If we allow OpenCL-style swizzles, `a.arg` for example (for elements <4, 1, 3> in order), we would suddenly have a `<f32, ..3>`, which may not be predeclared.

Ok, so we predeclare all primitive types, of all widths up to 256 bits, don't allow any wider vectors, and add them to the standard library, hoping it won't bloat things up, and use those names by default, well… suddenly we gained 144 types in the SIMD library that we have to keep track of in the compiler, and the only thing we gained was a slight reduction of syntax. (And that is before we someone wants Rust to run on a Xeon Phi that supports 512-bit vectors)

LLVM should be able to legalize all SIMD types (including ones not supported on x86, such as `<u8*, ..4>`) on all processors, so having additional ones defined shouldn't be a problem, but if people go bananas and start using 1024-bit vectors or something on x86 and LLVM complains, we might have to add a warning to the documentation of the limits.


Syntax for Shuffles
--------------------------------------------------------------------------------

For shuffles, I prefer the OpenCL accessor syntax `a.xyz` (swizzling, duplication _and_ nesting is allowed), but the more cumbersome OpenCL shuffle function `shuffle(a, (uint3)(0, 1, 2))` has uses as well (the second argument can be variable, so it can be used for sorting).

OpenCL and Clang supports both variants, GLSL supports only the first, GCC supports the only the latter, ICC supports none of them.

I think that supporting all the OpenCL accessors, and part of the GLSL ones (`rgba` for colour) seems like a good idea, and then adding a shuffle intrinsic later, if we feel that we need it.


Generic Vectors
--------------------------------------------------------------------------------

I think we should be able to support generic vectors, at least in element type. This would cut down on the number of functions that we have to implement in libstd, and could also reduce the amount of code in external libraries (lmath comes to mind.)

If we get associated items, we could make them generic in element count as well, which could be useful, since you could then define types like `<T, ..(16 / T::size)>`


Traits
--------------------------------------------------------------------------------

I think we need at least one trait that is automagically implemented on SIMD types, I'll call it `SIMD` for now. They should also have all the traits of their elements implemented, where the implementation is an _element-wise_ equivalent of the trait.

The `SIMD` trait should imply all functions that are specific to SIMD types, and in the future, it could also have the associated items `element_type` and `length`.


Functions
--------------------------------------------------------------------------------

I'm not listing the processor-specific intrinsics, listing them would be tedious, and I'll just assume that you're familiar with Neon/SSE if you want to use them.

I'll also not list functions available on the scalars, since they should just be implemented element-wise.

The following functions should be implemented for float vectors, because they are useful.

 - `fn cross(a:<T, ..{3,4}>, b:<T, ..{3,4}) -> <T, ..{3,4}>`: Standard vector product. The `w` component, if applicable, will be set to `0`.
 - `fn distance(a:<T, ..n>, b:<T, ..n>) -> T`: Euclidean distance.
 - `fn dot(a:<T, ..n>, b:<T, ..n>) -> T`: Scalar product.
 - `fn normalize(a:<T, ..n>) -> <T, ..n>`: Normalize to length 1.

The following _unsafe_ functions should be implemented for all vectors of length 2, 3, 4, 8 and 16,

 - `fn vload{n}(ptr:*T, offset:uint) -> <T, ..n>`: Loads a vector from `ptr + (offset * n)`.
 - `fn vstore{n}(ptr:*mut T, offset:uint, value:<T, ..n>)`: Stores a vector to `ptr + (offset * n)`.

These are _required_ since you cannot just cast a `*T` to a `<T, ..n>*` since the loads could become unaligned, which on many architectures could be quite nasty. These functions would take the necessary precautions, so that they can load from unaligned addresses.


Summary
--------------------------------------------------------------------------------

I want to be able to write `let x = <4.0f32, 2.0f32, 1.0f32, 9.0f32>;` and follow up with a `let y = x.normalize()` and other cool stuff, and I want it to be really fast and map down to good assembly when the LLVM optimizers are running.

I also don't want to have to trust the automatic vectorizers in LLVM, because I think they are unpredictable.