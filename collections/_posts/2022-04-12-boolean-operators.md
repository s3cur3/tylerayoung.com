---
title: "TIL about the differences between Elixir's boolean operators"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
image: "/assets/images/boolean-operators.png"
excerpt_full: true
---

TIL the difference between Elixir's "relaxed"/symbolic Boolean operators (`||`, `&&`) and their "strict"/word correspondents (`and`, `or`).

1. Relaxed versions do type coercion; strict require exactly `true` or `false`
2. Strict versions are allowed in guards

Relevant docs:

- Or:
    - [or/2](https://hexdocs.pm/elixir/main/Kernel.html#or/2)
    - [\|\|/2](https://hexdocs.pm/elixir/main/Kernel.html#%7C%7C/2)
- And:
    - [and/2](https://hexdocs.pm/elixir/main/Kernel.html#and/2)
    - [&&/2](https://hexdocs.pm/elixir/main/Kernel.html#&&/2)
