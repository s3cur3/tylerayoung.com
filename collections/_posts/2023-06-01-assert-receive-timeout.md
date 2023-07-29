---
title: "TIL about ExUnit's global assert_receive_timeout default config"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
excerpt_full: true
---

We've had a persistent problem at work with Elixir tests of Phoenix Channel interactions failing in CI only. The simplest push + reply you can imagine may, due to the vagaries of low-end, shared CI hardware, wind up taking a full second to receive a reply. 🥴

I'd been fixing these one at a time by adding a third timeout parameter to my assert_receive, assert_push, assert_reply, and assert_broadcast calls.

TIL you can instead just do:

```elixir
config :ex_unit, assert_receive_timeout: if(ci?, do: 2_000, else: 100)
```

(In [the main ExUnit docs](https://hexdocs.pm/ex_unit/main/ExUnit.html), search for `:assert_receive_timeout`.)

Phoenix LiveView 0.19.5 and later include my very modest [PR](https://github.com/phoenixframework/phoenix_live_view/pull/2655) to use this as the default timeout for LiveView's message-receiving test functions too (`assert_patch`, `assert_redirect`, `assert_push_event`, and `assert_reply`).


