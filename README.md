Why Rust and Web Assembly!
==========================


Low-Level Control with High-Level Ergonomics
--------------------------------------------
JavaScript Web applications struggle to attain and retain reliable performance. JavaScript's
dynamic type system and garbage collection pauses don't help. Seemingly small code changes can 
result in drastic performance regressions if you accidentally wanter off the JIT's happy path.

Rust gives programmers low-level control and reliable performance. It is free from the 
non-deterministic garbage collection pauses that plauge JavaScript. Programmers have control over
indirection, monomorphization, and memory layout. 


Small `.wasm` Sizes
-------------------
Code size is incredibly important since the `.wasm` must be downloaded over the network. Rust
lacks a runtime, enabling small `.wasm` sizes because there is no extra bloat included like a 
garbage collector. You only pay (in code size) for the functions you actually use.


Do *Not* Rewrite Everything
-----------------------------
Existing code bases don't need to be thrown away. You can start by porting your most performance-
sensitive JavaScript functions to Rust to gain immediate benefits. And you can even stop there if
you want to.


Plays Well With Others
----------------------
Rust and WebAssembly integrates the existing JavaScript tooling. It supports ECMAScript modules and
you can continue using the tooling you already love, like npm and Webpack.


The Amenities You Expect
------------------------
Rust has the modern amenities that developers have come to expect, such as:

- strong package management with `cargo`, 
- expressive (and zero-cost) abstractions,
- and a welcoming community! :)


Background and Concepts 
=======================
This section provides the context necessary for diving into Rust and WebAssembly development.


What is WebAssembly?
--------------------
WebAssembly (wasm) is a simple machine model and executable format with an [extensive specification](https://webassembly.github.io/spec/).
It is designed to be portable, compact, and execute at or near native speeds.

As a programming language, WebAssembly is comprised of two formats that represent the same 
structures, albeit in different ways:

1. The `.wat` text format (called `wat` for "WebAssembly Text") uses [S-expressions](https://en.wikipedia.org/wiki/S-expression), and bears 
some resemblance to the Lisp family of languages like Scheme and Clojure.

2. The `.wasm` binary format is lower-level and intended for consumption directly by wasm 
virtual machines. It is conceptually similar to ELF and Mach-O.

For reference, here is a factorial function in `wat`:

```
(module
    (func $fac (param64) (result f64)
        local.get 0
        f64.const 1
        f64.lt
        if (result f64)
            f64.const 1
        else
            local.get 0
            local.get 0
            f64.const 1
            f64.sub
            call $fac
            f64.mul
        end)
    (export "fac" (func $fac)))
```

If you're curious about what a `wasm` file looks like you can use the [wat2wasm](https://webassembly.github.io/wabt/demo/wat2wasm/) 
demo with the above code.


Linear Memory
-------------
WebAssembly has a very simple [memory model](https://webassembly.github.io/spec/core/syntax/modules.html#syntax-mem). A wasm module has access to a single "linear memory",
which is essentially a flat array of bytes. This [memory can be grown](https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-memory) by a multiple of the page
size (64K). It cannot be shrunk.


Is WebAssembly Just for the Web?
--------------------------------
Although it has currently gathered attention in the JavaScript and Web communities in general, wasm
makes no assumptions about its host evnironment. Thus, it makes sense to speculate that wasm will
become a "portable executable" format that is used in a variety of contexts in the future. As of
*today* however, wasm is mostly related to JavaScript (JS), which comes in many flavors
(including both on the Web and [Node.js](https://nodejs.org/)).


Tutorial: Conway's Game of Life
===============================
This is a tutorial that implement Conway's Game of Life in Rust and WebAssembly.


Who is this tutorial for?
-------------------------
This tutorial is for anyone who already has basic Rust and JavaScript experience, and wants to 
learn how to use Rust, WebAssembly and JavaScript together.

You should be comfortable rading and writing basic Rust, JavaScript, and HTML. You definitely do
not need to be an expert.


What will I learn?
------------------

- How to set up a Rust toolchain for compiling to WebAssembly. 
- A workflow for developing polyglot programs made from Rust, WebAssebly, JavaScript, HTML,
and CSS.
- How to design APIs to take maximum advantage of both Rust and WebAssembly's strengths and 
also JavaScript's strengths.
- How to debug WebAssembly modules compile from Rust.
- How to time profile Rust and WebAssembly programs to make them faster.
- How to size profile Rust and WebAssembly programs to make `.wasm` binaries smaller and faster
to download over the network.


The results of this tutorial are all in the code with my appended comments.

[This](https://rustwasm.github.io/docs/book/introduction.html) is the original source of the project.


Quick note: 
-----------
When designing an interface between WebAssembly and JavaScript, we want to optimize for the following
properties:

	1. **Minimizing copying into and out of the WebAssembly linear Memory.** Unnecessary copies
     	   impose unnecessary overhead.
	
	2. **Minimixe serializing and deserializing.** Similar to copies, serializing and deserializing
	   also imposes overhead, and often imposes copying as well. If we can pass opaque handles to
	   a data structure -- instead of serializing it on one side, copying it into some known location
	   in the WebAssembly linear memory, and deserializing on the other side -- we can often reduce
    	   a lot of overhead. `wasm_bindgen` helps us define and work with opaque handles to JavaScript
	   `Object`s or boxed Rust structures.

As a general rule of thumb, a good Javascript<->WebAssembly interface design is often one where large,
long-lived data structures are implemented as Rust types that live in the WebAssembly linear memory, and
are exposed to JavaScript opaque handles. JavaScript calls exported WebAssembly functions that take these
opaque handles, transforms their data, perform heavy computations, query the dtata, and ultimately 
return a small, copy-able result. By only returning the small result of the computation, we avoid copying
and/or serializing everything back and forth between the JavaScript garbage-collected heap and teh 
WebAssembly linear memory.

