+++
date = '2026-08-13T11:00:00+09:00'
draft = false
title = "I'm winding down ClojureWasm"
summary = 'I stopped developing ClojureWasm on August 12, 2026; v1.10.1 is the final release. What I shut down, why I stopped, why zwasm keeps going in its own organization, and what half a year of AI-assisted work on a language runtime actually taught me.'
tags = ['clojure', 'webassembly', 'zig']
+++

Two months ago I wrote about building a Clojure implementation from scratch,
with a lot of help from AI.

[I built ClojureWasm, a Clojure runtime that runs WebAssembly](https://chaploud.github.io/ChaosCloud/posts/clojurewasm-introduction/)

But I've decided to wind ClojureWasm down. Development stopped on August 12,
2026, and v1.10.1 is the final release.

## What I shut down

- [ClojureWasm](https://github.com/clojurewasm/ClojureWasm)
  - Not archived. The README now carries a notice saying the project is no
    longer maintained.
- [zwasm](https://github.com/zwasm/zwasm)
  - Transferred to [its own organization](https://github.com/zwasm).
  - [jtakakura](https://github.com/jtakakura), who contributed a great deal to
    it, and I are continuing to maintain it together.
- The other repositories — the ones showing ClojureWasm in use
  - All archived.

## Why I decided to wind it down

I lost confidence that ClojureWasm had a future where it actually gets used.

I still think the idea itself was a good one: stay in the Clojure world, and
reach for what other languages have already built by going through a Wasm FFI.

But taking compatibility seriously meant an endless queue of Java interop to
implement, and chasing performance on top of that. Between them, the project
started to crowd out ordinary life. And as far as being a lightweight Clojure
runtime goes, there are already several out there with far better Java interop.
It started to look like something that would get done whether or not I was the
one doing it.

## I decided to keep zwasm

I wrote zwasm so that ClojureWasm would have something to run Wasm with. It is
independent of Clojure: a Wasm runtime and library, small, with full support up
through Wasm 3.0 and WASI 0.3, and a JIT. That one still has room to grow, and
it has been getting genuinely solid, so I decided to keep maintaining it in a
separate organization.

## What I learned, and how I honestly feel

First, the reason I could build OSS at this scale at all is AI. A few years
ago, one person getting this far in half a year would have been impossible.

At the start I was involved in a lot of it — researching, designing, putting
guardrails in place. But partway through, the AI reached a state where it could
run on its own, and the result was that I stopped being able to follow the code
down to the fine detail. I was also learning less. Back when I was reading
around, puzzling over things, debugging and writing code by hand, the process
alone burned itself into memory. Much of that opportunity is gone now.

On the other hand, I did accumulate real knowledge: separating responsibilities,
separating layers, design in general, and development loops that make use of AI.
It has been paying off in my day job.

I learned how hard OSS work is. ClojureWasm had almost no users yet, so there is
surely plenty I still cannot see. Even so — the maintainers who keep going while
asking themselves "does this really mean anything?" and "is this worth
continuing when it eats into my real life and my money?" — I think those people
are remarkable.

## Going forward

ClojureWasm stays unarchived; only maintenance stops. You are free to fork it,
modify it, and release it as your own product. You don't need to contact me. It
has been published under EPL-2.0 from the start, so just follow those
guidelines.

Clojure itself is a wonderful language, and the community still feels like it
has real heat in it. I intend to keep using it, both at work and in my own
products.
