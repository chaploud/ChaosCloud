+++
date = '2026-05-25T23:17:13+09:00'
draft = false
title = 'Hello, Chaos Cloud'
summary = 'Welcome to Chaos Cloud — a blog on language runtimes, software craft, Clojure, Zig, and learning.'
tags = ['meta', 'announcement']
+++

Welcome to **Chaos Cloud** — a place where I jot down what I learn while
poking at language runtimes, building software, and trying to teach the
ideas underneath.

## What you can expect here

- **Language implementation** — interpreters, compilers, garbage
  collection, the small print of evaluation order.
- **Clojure** — REPL-driven design, data orientation, and the libraries
  that come with the territory.
- **Zig** — low-level work where every byte and every allocation is on
  the page.
- **Software craft** — tools, workflows, and habits that survive past a
  single project.
- **Education** — explaining the hard parts so someone newer can read
  it without giving up.

## Why "Chaos Cloud"

Chaos because real systems are messy and the best ideas live near the
edge of what we understand. Cloud because notes float, drift, and
sometimes condense into something useful.

```clojure
(defn welcome [reader]
  (str "Hello, " reader ". Glad you're here."))
```

```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello, Chaos Cloud\n", .{});
}
```

More posts soon.
