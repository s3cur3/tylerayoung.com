---
title: "Open Sourcing Jump's Credo Checks"
layout: post
authors: ['tyler']
categories: ["Programming", "Functional Programming", "Elixir"]
excerpt_full: true
---

A good portion of my job on the developer experience team at Jump is focused on improving code quality, and writing Credo[¹](#1-what-is-credo) checks is a significant part of that.

Today, Jump is open sourcing all the Credo checks we've built internally which might be of value to the rest of the community. [`Jump.CredoChecks`](https://hex.pm/packages/jump_credo_checks/) encodes a lot of very opinionated (hi, have you met me? 👋) knowledge about ways that both production code and tests can go wrong, and how to fix them. We run each of these (in addition to many more checks that are only of interest to us internally, like things that ensure we're using the right UI components from our in-house design system) on every PR and block merges whenever one fails.

As there are 14 checks in the initial release, let me call out a few of my favorites. (Of course, you can check the [quite thorough docs](https://hexdocs.pm/jump_credo_checks/) for info on the rest.)

<!--more-->

## My favorites

### `AssertElementSelectorCanNeverFail`

This code has slipped past me too many times in code review:

```elixir
test "my LiveView", %{conn: conn} do
  {:ok, view, _html} = live(conn, ~p"/")
  assert element(view, "button", "Log in")
end
```

The problem there is that `LiveViewTest.element/{2,3}` is *always* truthy; when the element is missing, it returns the empty list. Instead, you need to use `LiveViewTest.has_element?/{2,3}`.

### `AvoidFunctionLevelElse`

A refactor once accidentally turned this:

```elixir
def my_fun(arg) do
  with {:ok, whatever} <- other_fun(arg),
       ... do
     whatever
  else
    {:error, reason} -> handle_error(reason)
  end
end
```

..into this:

```elixir
def my_fun(arg) do
  other_fun(arg)
else
  {:error, reason} -> handle_error(reason)
end
```

Did you know you could have an `else` at the top level of a function? I certainly didn't. It exists to complement the top-level `rescue`, and it runs only when the function does _not_ raise an exception. However, in the example above, the function will crash any time `other_fun/1` doesn't return an `:error` tuple (raising a `TryClauseError`).

It's legal syntax, but it's _almost_ never actually useful.

### `DoctestIExExamples`

If I see `iex>` in a `@doc` or `@moduledoc`, it leads me to believe that code is actually tested and therefore more reliable than less structured examples. This Credo check guarantees that by checking that the corresponding test file actually invokes `doctest` on the module.

### `LiveViewFormCanBeRehydrated`

The rules around LiveView form state recovery are simultaneously simple and incredibly easy to screw up. This adds a small layer of protection by at least guaranteeing a couple prerequisites for form recovery are in place: an ID and a `phx-change` handler.

### `TopLevelAliasImportRequire`

LLMs loooove to stick `alias`, `import`, and `require` statements in the body of a function. This ensures they only live at the module root.

### `VacuousTest`

Warns about tests which don't appear to actually exercise any production code. LLMs like to generate these, especially if the thing they were supposed to be testing would be hard.

### `WeakAssertion`

It pains me to see an assertion like `refute is_nil(...)`. "Oh good," I think, "we've eliminated one out of an infinite number of possible values."

A similar gem: `assert is_binary(...)`. Is it the empty string? Does it have any particular content? Can we say *nothing* else about the string except that it exists? Really?!

This check encourages better assertions; tell the reader something *about* the value!

## Let me know what you think

If you try the checks, reach out [on BlueSky](https://bsky.app/profile/tylerayoung.com) or [on Mastodon](https://fosstodon.org/@tylerayoung) and let me know what you think—the good, the bad, and the ugly. I hope they're as useful to you as they have been to our team, but if not, I'd love to hear how we can improve them.

## Footnotes

### 1. What is Credo?

[Credo](https://github.com/rrrene/credo) is the most popular tool for doing static analysis in Elixir.
