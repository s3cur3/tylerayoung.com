---
title: Write More Pure Functions
date: 2022-03-16
layout: post
authors: ['tyler']
categories: ["Programming", "Functional Programming"]
image: "/assets/images/san-diego-2018/2018-12-23-12.14.46-explosive-water.jpg"
---

Let me begin by saying: you probably shouldn’t read this. You have a limited amount of time, and you’re probably better served reading John Hughes' seminal [Why Functional Programming Matters](http://www.cse.chalmers.se/~rjmh/Papers/whyfp.pdf). But this is my perspective on why pure functions are important—the stuff I feel *viscerally* day to day, not academically. It's my exhortation to think about this stuff constantly in the course of programming, even in languages that don’t strictly enforce it—maybe especially in those languages.

You can, and should, extend your [functional core](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell) ever outward toward the edges of your system.

## An intuitive definition of pure functions

If you've been around programming for awhile, you're probably familiar with the definition of a pure function as one that fulfills two criteria:

1. Its output depends only on its arguments (plus maybe global constants).
2. It has no side effects.

A more intuitive definition<a href="#footnote-1" id="footnote-1-return">¹</a> is: A function is pure if and only if it could be replaced by a (potentially infinitely large!) lookup table. For instance, the function “string contains” could, in theory, be replaced by a lookup table of all possible strings, crossed with all their possible substrings; then to find out if “foo” contains “bar,” you could look up the entry for “foo” and see whether “bar” is in its list of substrings.

In contrast, the function “database column contains string” is not pure, because it’s not even possible in theory to provide a lookup table for the state of your database—you have to ask the database! Likewise, “write this string to a database row” isn’t pure because of course it “does stuff” rather than “returns stuff.”

Interestingly, this litmus test explains concisely why functions that depend on randomness or the current time are not pure, even if they are free of side-effects; you have to actually make the call to find out what you’ll get this time.

## Why try care about pure functions?

Pure functions have a few important properties from my perspective:

### 1. They make testing not absolutely suck

I *love* automated testing. A strong test suite is one of the things that’s most important in my day-to-day happiness as a software developer. But testing components that require a bunch of hidden state to be set up in advance is awful. These tests end up as “write-only” code—nobody else (including your future self) will really understand how or why they work, but as long as the tests pass, “I guess it’s okay,” right?

Testing pure functions, on the other hand, is delightful. Even when they require a lot of setup, the setup is of the form “create a bunch of values, then pass them as arguments.” If your future self has a question about why the function needs a particular value, you can, well… look at how it’s used within the function. There’s no mystery. Likewise, if the test setup offends your coworker's sensibilities because it seems overwrought, again, they can look at how the data is used and either split up your function or agree with you that it’s the best option. That brings me to my second point:

### 2. Refactoring is trivial

A well-tested pure function can be tweaked, split up, torn apart, turned inside out, and reassembled while never worrying about breaking other stuff. They are resilient to changing in whatever way your business, domain model, or personal tastes require. 

This is in direct contrast to code with side effects, or code that depends on global state. The most universal experience in programming is to be bitten by hidden dependencies while maintaining code—you removed a call to system A from system B, and now suddenly system C is inexplicably broken. These kinds of bugs are still possible in a system composed of pure functions (for instance, you forgot that some function required a sorted list and you gave it an unsorted one), but in my experience they are just vastly more rare—two orders of magnitude more rare, maybe.

### 3. Your future self will thank you

Because pure functions are easier to understand, test, and (safely) change, they’re some of the only code I've written that I’ve come back to a year later and thought “well done, past self.” I've had this experience so many times that "mostly pure functions, with comprehensive tests" has become [the fast path for code review](https://twitter.com/TylerAYoung/status/1499927366856024064) for me. Give me those two things and I don't really care about the other stuff people put on the "good code" pedestal.

In reflecting on this, it was kind of weird to me that 2 of the 3 things that make me feel strongly about this deal with the future, not the present. But that’s just the thing, isn’t it? It’s easy enough *today* to write a [Big Ball of Mud](https://en.wikipedia.org/wiki/Big_ball_of_mud) that does more or less the right thing... the difficulty really only sets in later when you need to change things. As Titus Winters [says](https://twitter.com/TitusWinters/status/1081172434106175489), software engineering is programming integrated over time. If the “over time” bit doesn’t apply to you, go wild, do whatever, there’s no downside. But if you’re going to have to maintain the code, those maintenance costs will probably dwarf the initial development; it makes sense to minimize them from the start.

And hey, if six months down the line, you decide this purity stuff is bullshit, it’s easy to add side effects and global state. 🙃 (Much more difficult to go the other direction, though!)

## Problems with purity

There are two things that give me pause in saying “go forth, and write pure functions”; both are more consequences of poor ecosystem support than anything else.

### 1. Passing more data than is needed to a function

It’s easy to fall into a bad habit of saying “this function needs to only depend on its inputs? Fine, I’ll pass it the entire state of the program!”

This is kind of trivial, because the fix is, well... just don’t do that. But it’s also the case that languages don’t make things easier here. [Lenses](https://medium.com/@dtipson/functional-lenses-d1aba9e52254) are a possible solution, but they’re hard to learn or teach—the only people I know who feel really comfortable with them are hardcore Haskell nerds, and frankly if that's the requirement, it will remain out of reach for most developers forever.

I’m actually quite fond of Typescript’s `Pick<Type, Keys>` helper here. It lets you can say things like “all this function really needs are these three keys from some bigger data structure.” This has a few nice features:

1. You get a compiler error if that data structure ever drops or renames those keys.
2. Your tests only need to fill in a few fields rather than constructing the whole state of the world.
3. Clients are allowed to pass _more_ data—it just remains "invisible" to your function's implementation.

### 2. Purity is fragile<a href="#footnote-2" id="footnote-2-return">²</a>

Most programming languages have terrible support for expressing the idea that a function is pure. Even the ones that can express this generally can't enforce that it maintains that property. As a consequence, it’s very easy for someone making a tactical change to a function deep in your stack to accidentally introduce some dependency on global state or some important side effect. When that happens, _bam_! Suddenly the purity property you depended on far, far from that function definition has been invalidated. This is the kind of thing that's very easy to miss in code review too, because of the narrow lens through which most people tend to review changes (myself included, unfortunately).<a href="#footnote-3" id="footnote-3-return">³</a>

C++ (surprisingly?) sort of has an annotation for this: you can [mark a function as `__attribute__((const))`](https://stackoverflow.com/questions/2798188/pure-const-function-attributes-in-different-compilers#2798511) to swear to the compiler it’s a pure function.<a href="#footnote-4" id="footnote-4-return">⁴</a> While this is a good start, it exists primarily as a performance optimization, and in my experience it’s poorly supported by the tooling—you get little to no help from the compiler if you violate the purity gaurantee in the implementation.

What I’d really like is for every language to have a keyword or something that says “this is not just any function I’m declaring, it is a _pure_ function; I guarantee it will only depend on its inputs, and any function it calls will also declare _itself_ pure, otherwise the program won't compile.” (I've played around with writing a macro for this in Elixir—something like `defpure`—but not gotten very far.)

## Go forth and... you get it.

The longer I program professionally, the more important it's become to me to ensure not only that the code I write is correct _today_, but that it will remain correct in the face of future changes by people with less context than I had when I wrote it. (This includes future me.) "More pure functions" is the most effective way I've found to do this.

If you aren't convinced, I can't recommend highly enough that you play around with a language where purity is enforced. [Elm](https://elm-lang.org) is a gentle, yet powerful introduction to this world. It will change your perspective on writing nominally imperative languages.

If you see something I missed, hit me up on Twitter [@TylerAYoung](https://twitter.com/TylerAYoung/).

## Footnotes


<a href="#footnote-1-return" id="footnote-1">¹</a>This is an idea I heard from Richard Feldman’s outstanding “[Functional Programming for Pragmatists](https://www.youtube.com/watch?v=3n17wHe5wEw).” Feldman himself [attributed](https://twitter.com/rtfeldman/status/1492907321827241988) it to Kris Jenkins, who in turn [attributed](https://twitter.com/krisajenkins/status/1492911584510984196) it to maybe *Let Over Lambda*. I couldn’t track down a citation, but Jenkins may have been inspired by the “[Lisp is Not Functional](https://letoverlambda.com/textmode.cl/guest/chap5.html)” section of that book.

<a href="#footnote-2-return" id="footnote-2">²</a> This does not apply to purely functional languages like Elm or Haskell, where you have to do some truly horrifying hacks to make a function fail the purity test. The percentage of developers working in that environment, though, rounds to zero. 😕


<a href="#footnote-3-return" id="footnote-3">³</a> The whole analogy to lookup tables above is reminiscent of [memoization](https://en.wikipedia.org/wiki/Memoization). This is, in my experience, a trap... not because of problems with the concept of memoization, but because of a team's incredible propensity to invalidate the function purity invariant. I've seen 4 separate codebases, in 3 separate organizations where a clever, bright eyed developer said “this function is pure, let’s memoize it!”... only to be bitten by a subtle impurity being introduced within six months and discovered via a bug report against production. (Sometimes that bright eyed programmer has been me. 😬)

<a href="#footnote-4-return" id="footnote-4">⁴</a> In the fine tradition of C++ screwing up naming, this means something different than `__attribute__((pure))`. It also [doesn't work the way you might expect]({% post_url 2020-07-02-warning-__attribute__const-does-not-mean-function-purity %}).

