+++
date = '2026-06-23T12:00:00+09:00'
draft = false
title = 'I built ClojureWasm, a Clojure runtime that runs WebAssembly'
summary = 'ClojureWasm is a Clojure implementation written in Zig and Clojure. Most existing Clojure code runs on it, and you can :require a .wasm module like a namespace and call it with plain Clojure data.'
tags = ['clojure', 'webassembly', 'zig']
+++

I write full-stack Clojure web apps for a living — frontend, backend, and the
mobile apps too. Clojure is one of those languages that gets harder to put down
the longer you use it. It's fairly niche, especially in Japan, but I fell for it
anyway, and eventually I did the thing you do when you fall too hard for a
language: I wrote my own implementation of it.

https://github.com/clojurewasm/ClojureWasm

Clojure already runs in a lot of places. There are implementations that sit on
top of Java, .NET, JavaScript, Dart (Flutter), and C++. There's also
[Babashka](https://github.com/babashka/babashka), a runtime built for fast CLI
scripting that a lot of the ecosystem leans on.

ClojureWasm started as a side project, mostly out of curiosity and a desire to
understand how this stuff actually works. Before the current repo there were
three or so earlier attempts that I built and threw away. By the time I started
this one I had enough design sense and scar tissue to begin again from scratch,
and it finally turned into something usable. The result is a small single
binary that starts fast, runs most existing Clojure code as-is, and can execute
WebAssembly compiled from other languages. I didn't set out to land in that
exact niche; it's more something I noticed I'd built along the way. This post
walks through a few of the things you can do with it.

## Overview

![ClojureWasm logo](clojurewasm_logo.png)
{style="width:50%;margin-inline:auto;"}

ClojureWasm is a Clojure implementation written in Zig and Clojure itself. I
didn't want the usual "only part of Clojure works" compromise, so the goal is
for nearly all existing code to run. Reimplementing Java and the whole JVM is
off the table, obviously, so the approach is to port the Java interop that real
Clojure libraries actually reach for, one piece at a time, with equivalent
behavior.

The part I'm most excited about: if you compile a program written in some
other language to Wasm, you can run it from Clojure code. So you get to write
Clojure and still pull in another language's ecosystem when you need it. It
already works reasonably well, and you can try it in
[this Playground](https://cw-playground.fly.dev/).

## Running Clojure code

Most Clojure features behave the way you'd expect.

### Threading macros (`->` / `->>`)

The familiar functional pipeline for shuffling data around.

```clojure
(-> {:name "Alice" :age 30}
    (assoc :role :admin)
    (update :age inc))
;; => {:name "Alice", :age 31, :role :admin}

(->> (range 10)
     (filter even?)
     (map #(* % %))
     (reduce +))
;; => 120
```

### State with `atom`

An `atom` is a handy single source of truth for mutable state. `swap!`
applies a function to the current value and stores the result.

```clojure
(def counter (atom 0))
(swap! counter inc)
(swap! counter + 10)
@counter
;; => 11
```

### Concurrency with `future` / `promise`

`future` runs work on another thread, and `@` (deref) blocks until you have a
result. These are real threads, so you can actually run heavy work in parallel.

```clojure
;; run two computations in parallel and add the results
(let [a (future (reduce + (range 1000000)))
      b (future (reduce + (range 1000000)))]
  (+ @a @b))
;; => 999999000000

;; pass a value between threads with promise
(let [p (promise)]
  (future (deliver p 42))
  @p)
;; => 42
```

### Java interop

Existing Clojure libraries call Java classes and methods constantly, so
ClojureWasm adds the commonly used interop in Zig with matching behavior.

```clojure
(Math/sqrt 144)                          ;; => 12.0
(.toUpperCase "clojure")                 ;; => "CLOJURE"
(Integer/parseInt "42")                  ;; => 42
(str (java.time.LocalDate/of 2026 6 23)) ;; => "2026-06-23"
```

## Calling Wasm

### A simple Wasm call

Here's a tiny Wasm module, `add.wasm`, that does nothing but add two integers.
It exports a single function called `add`.

```scheme {title="add.wat"}
(module
  (func (export "add") (param i32 i32) (result i32)
    local.get 0
    local.get 1
    i32.add))
```

Turn it into `add.wasm` with `wat2wasm add.wat -o add.wasm`. From Clojure you
load it and call the function by name.

```clojure
(wasm/call (wasm/load "add.wasm") "add" 40 2)
;; => 42
```

If all you need is numbers in and numbers out, that's enough. In real code you
usually want to pass something with a bit more shape: lists, strings, records.
That is where the **WebAssembly Component Model** comes in.

### Running other languages' code with the Wasm Component Model

A WebAssembly component carries the type information for the functions it
exports, in a format called **WIT**. Below is a small piece of work that takes
a record holding a list of numbers and a label, processes it, and hands it
back.

```rust {title="world.wit"}
package zwasm:typedtest;

interface types {
  record payload {
    xs: list<u32>,
    label: string,
  }
}

world typed-test {
  use types.{payload};
  export process: func(input: payload) -> result<payload, string>;
}
```

I'll write the implementation in Rust, using the
[`wit_bindgen`](https://github.com/bytecodealliance/wit-bindgen) crate.

```rust {title="src/lib.rs"}
wit_bindgen::generate!({ path: "wit", world: "typed-test" });

struct Component;

impl Guest for Component {
    fn process(input: Payload) -> Result<Payload, String> {
        let mut xs = input.xs;
        xs.push(xs.iter().sum()); // append the sum to the end
        // tack a "!" onto the label
        Ok(Payload { xs, label: format!("{}!", input.label) })
    }
}

export!(Component);
```

Build it as a WebAssembly component:

```sh
cargo build --target wasm32-wasip2 --release   # -> typed_payload.wasm
```

Then from Clojure you `:require` the resulting `.wasm` as if it were a
namespace. The exported `process` function shows up as an ordinary Clojure
function, `tp/process`.

```clojure
(ns my.app
  (:require ["typed_payload.wasm" :as tp]))

(tp/process {:xs [3 4 5] :label "data"})
;; => {:xs [3 4 5 12], :label "data!"}
```

The sum of `[3 4 5]`, which is `12`, got appended, and the label came back with
a `!` on it.

What I like here is that the arguments and the return value are **just Clojure
data**. A WIT `record` becomes a map, a `list` becomes a vector, a `string`
becomes a string. You don't write any of the tedious glue to convert types
between Rust and Clojure.

The argument names from the WIT also survive into Clojure metadata, so the
function looks like any other Clojure function.

```clojure
(:arglists (meta #'tp/process))
;; => ([input])
```

### Why this is nice

Clojure has always been a language that runs on a host and calls that host's
API. The JVM version calls Java, ClojureScript calls JavaScript, ClojureDart
calls Dart. Each one is tied to its host language.

With a WebAssembly component, the host isn't a specific language; it's a
language-agnostic `.wasm`. And because the component carries its own type
information in WIT, the Clojure side only has to read the `.wasm` to learn what
functions exist and what types their arguments and return values are. That's
where you get to skip writing the glue code by hand.

## Reflections

It took three or four full rewrites, each one thrown away, before ClojureWasm
got to a state I was willing to publish. Take something that looks like a
surface-level feature: wanting it to run faster, say, or wanting proper error
traces. If you don't set the conventions at the design level from the start, the
codebase fills up with ad-hoc patches. I spent a lot of effort on the design
side: keeping the direction of dependencies linear, arranging things so most
features have a single place where you change them. There are far too many small
decisions to write down here, but I came away convinced that no amount of
upfront analysis beats just building the thing once. It's a way of working that
only really opens up now that autonomous AI development loops exist.

## Wrapping up

A few of the things ClojureWasm can do:

- A lot of existing Clojure code already runs.
- You `:require` a `.wasm` from Clojure like a namespace.
- Arguments and return values pass back and forth as plain Clojure data: maps,
  vectors, strings.

Writing a language implementation feels like building a small world, and I find
it genuinely fun. Building it reminded me all over again how good both Clojure
the language and WebAssembly the technology really are.

## Postscript

The Wasm engine inside ClojureWasm,
[zwasm](https://github.com/clojurewasm/zwasm), is a parallel project of its own,
and it was plenty of work in its own right. You can use it independently as a
Wasm runtime, library, and CLI.
