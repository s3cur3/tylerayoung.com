---
title: "TIL about Elixir's <code>System.unique_integer/1</code>"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
excerpt_full: true
---

TIL about [`System.unique_integer/1`](https://hexdocs.pm/elixir/System.html#unique_integer/0), which guarantees you an integer that it hasn't (yet) handed out in the current runtime. 😍

Pass in `[:positive, :monotonic]` as the argument to get a unique, increasing number—useful for ordering in tests to know at a glance that one entity was created before another:

```elixir
user1 =
  AccountsFixtures.user_fixture(%{
    email: "u#{System.unique_integer([:positive, :monotonic])}@example.com"
  })

user2 =
  AccountsFixtures.user_fixture(%{
    email: "u#{System.unique_integer([:positive, :monotonic])}@example.com"
  })
```

(I'd started writing my own GenServer to do this, but gave up quickly trying to handle a bunch of edge cases. 😅)

