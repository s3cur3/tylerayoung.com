---
title: "Don’t Write Exploding Streams"
layout: post
authors: ['tyler']
categories: ["Programming", "Functional Programming", "Elixir"]
#image: "/assets/images/san-diego-2021/img_2701-edited.jpg"
#image: "/assets/images/rogers/img_2940-edited.jpg"
image: "/assets/images/stream-4-3.jpg"
excerpt: "Suppose you have a `Stream` of values that you want to insert into the database. You can do this easily in many modern programming languages, but here's what it might look like in Elixir:"

---

## Motivation: Why streams are great

Suppose you have a `Stream` of values that you want to insert into the database. You can do this easily in many modern programming languages, but here's what it might look like in Elixir:

```elixir
Enum.each(my_stream, fn item ->
  item
  |> MyModule.do_some_transformation()
  |> MyDatabaseSchema.insertion_changeset()
  |> Repo.insert()
end)
```

The great thing about this—the thing that makes streams such a powerful concept!—is that the `Stream` is totally interchangeable with any other enumerable data structure. You could replace `my_stream` in the example with `my_list` or `my_set` or [`my_array`](https://github.com/Qqwy/elixir-arrays) and nothing else about the algorithm would have to change. The stream will Just Do the Right Thing™, but with the added benefit of not having to load the whole data set into memory at once.

## Poisoning the well

Now suppose you have a `Stream` that can fail for mundane reasons—network errors, file not found, malformed XML, etc.<a href="#footnote-1" id="footnote-1-return">¹</a> In the happy path, it streams values of some struct `S`, and you insert them into the database just like in the example above.

The question is: **how do we handle errors in the stream?**

You could be forgiven for wanting your `Stream` to be fully interchangeable with a `list` of `S`. In that case, you really only have maybe three options; you can either

1. silently eat errors (stopping the stream),
2. produce some sort of sentinel value in the stream when you encounter an error, or
3. throw an exception.

All of these are bad options, but the first two are *so* egregious that I don't feel like I need to argue against them. The third, though, is a siren song I've seen draw in many an API creator.

Using an exception in this case makes your stream prone to "exploding." Quite unlike a list or set of `S`, merely trying to read a value might cause the process to crash. Not only is this not clear from the interface or typespec, but it necessitates that the stream be handled like a ticking timebomb. The whole promise of the stream was "it's interchangeable with a list," but there's no such thing as a list of "maybe materializable values."<a href="#footnote-2" id="footnote-2-return">²</a> 

## Back to basics

Producing a `Stream` of `S` in this case is fundamentally the wrong type. You can't do it. You can only either produce a stream of *explosive* `S`s, or another type entirely.

Suppose we're designing an XML parser. Parsing is, by its nature, problematic and prone to failure. We have to decide from the outset how we'll handle errors. Should we:

- Keep all values up to the error, then abort?
- Discard the error, then keep going if possible?
- Abort the whole process if there's any error?

The key here is to recognize that as the designer of the XML API, we don't know how clients should handle errors! We have *way* less information than they do about what's best for their use case. Maybe they want to be strict, maybe they want to be permissive. If we throw an exception, we don't give them the opportunity to make that choice. (And, of course, we may cause a crash because it wasn't clear from our interface that this was "a thing" in the first place.)

Now, we're responsible API designers. We don't want to treat error handling as an afterthought. How *should* we design our API? The clearest way, to my mind, is to make our stream produce a result tuple:

- `{:ok, parsed_value}` on success
- `{:error, explanation}` on error

For errors that are recoverable, this lets clients decide how strict they want to be; for errors that our API can't recover from, at least we made it explicit to the client code that the error states are something they'll need to account for.

Clients won't be able to just treat our stream like a list of structs—they'll have to make a decision about how to handle errors. But that's a feature, not a bug; it's the nature of what they're trying to do! Hiding the error from the interface seems initially convenient, but it's ultimately a footgun. Please don't provide APIs for blowing off users' toes.

## Footnotes

<a href="#footnote-1-return" id="footnote-1">¹</a> I heard Phil Nash recently [describe](https://www.youtube.com/watch?v=9ngflAGoreM&t=25m43s) these as "disappointments" in contrast to "exceptions," making the point that these are not actually *exceptional* situations at all.

<a href="#footnote-2-return" id="footnote-2">²</a> God help me, as I write this I realize you *could* produce such a thing in some languages. Dear reader, I'm begging you, do not.


