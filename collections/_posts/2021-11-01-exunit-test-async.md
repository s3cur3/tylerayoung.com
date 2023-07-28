---
title: "TIL Elixir always runs tests within a test module synchronously"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
excerpt_full: true
---

TIL I totally misunderstood what ExUnit's `async: true` does.

I thought it made ExUnit run the tests in *this* module/file concurrently.

In fact, it *always* run tests in this module synchronously, but `:async` schedules them in parallel with tests in *other* files.

Imagine that... [reading the docs](https://hexdocs.pm/ex_unit/ExUnit.Case.html) teaches you things. It's right there at the top. 😅

> `:async` - configures tests in this module to run concurrently with tests in other modules. Tests in the same module never run concurrently. It should be enabled only if tests do not change any global state. Defaults to false.
