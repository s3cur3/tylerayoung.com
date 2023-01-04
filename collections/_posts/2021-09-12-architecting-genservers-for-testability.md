---
title: Architecting GenServers for Testability in Elixir
layout: post
authors: ['tyler']
categories: ["Programming", "Functional Programming", 'Elixir', "Conference Talk Summary"]
image: "/assets/images/union-station-reflection.jpg"
---

\[Post-publication update: This post turned into a conference talk at ElixirConf 2021, embedded here for posterity.\]

<p><iframe width="480" height="299" src="https://www.youtube.com/embed/EZFLPG7V7RM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></p>

This post is an attempt to lay out my philosophy on how we should be designing GenServers in Elixir. It says “… for testability” in the title, and we’ll focus on that, but there’s also a lot here on deciding what and how to test, as well as thinking about what the testing can tell us about how the rest of the system will use our module.

I draw a lot of inspiration here from lessons that the object-oriented community has learned the hard way. As a recovering C++ developer, I spent a long time thinking about how to test object-oriented systems, and I came along for the ride over the last decade or so as C++ and other OOP-y languages (yes, I know, C++ is “multi-paradigm,” don’t @ me) drew a ton of lessons from functional programming. I’ve seen how those ideas improved the design of our classes and object-oriented systems.

But first, you’re probably wondering:

What could OOP possibly teach Elixir developers?
------------------------------------------------

After all, if Elixir is a functional language, and C++ took a ton of inspiration from functional languages, what could C++ give _back_?

I’m going to say something that some folks aren’t ready to hear, but it’s true:

### Stateful Elixir processes (like GenServers) are sort of like objects

I know. I can _hear_ the angry clacking of keys already. “Well actually, GenServers are implemented as a recursive receive loop, where the state gets passed from one invocation to the next.”

Sure, that’s true. Under the hood, GenServers are pure-ish functions. But for _users_ of them, what’s the difference between this bit of Python:

```python
my_user = User("Tyler", "abcd-efgh-1234-5678")

# Prints "Hi, I'm Tyler"
my_user.say_hello()

# Prints a memory address:
# <__main__.User object at 0x103e4aeb0>
print(my_user)

# Prints the data:
# {'name': 'Tyler', 'id': 'abcd-efgh-1234-5678'}
print(my_user.__dict__)

# Hi, I'm Tyler Young
my_user.set_name("Tyler Young")
my_user.say_hello()
```

And this bit of Elixir:

```elixir
my_user = User.start_link("Tyler", "abcd-efgh-1234-5678")

# Prints "Hi, I'm Tyler"
User.say_hello(my_user)

# Prints a process ID
IO.inspect(my_user)

# Prints the state data:
# %{name: "Tyler", id: "abcd-efgh-1234-5678"}
IO.inspect(:sys.get_state(my_user))

# Hi, I'm Tyler Young
User.update_name(my_user, "Tyler Young")
User.say_hello(my_user)
```

To my mind, in day-to-day dev, this is a distinction without a difference (at least until you start [scheduling too much work in the GenServer](/2021/12/02/shooting-yourself-in-the-foot-with-genservers/) and shoot yourself in the foot).

Of course, you should be thinking way harder about creating a process than an object in an OOP language. Processes serve as [“atomic units of failure domains”](https://news.ycombinator.com/item?id=23255039); they also have a much higher cost than an object (Saša Jurić’s [To Spawn or Not to Spawn?](https://www.theerlangelist.com/article/spawn_or_not) is a classic here). To ignore this is to fall into the trap of writing Ruby in Elixir. All the same, GenServers sure do share a lot of the properties of objects.

(Indeed, actors & message passing are actually _more_ in line with what Alan Kay, one of the creators of Smalltalk, had in mind for OOP than the more rigid model that came out of C++ or Java.)

None of this is a criticism of Elixir—after all, your state has to live somewhere! But I would like to push back against the idea that because Elixir is “a functional language” (whatever that means), we don’t have anything to learn from OOP languages. Instead, I’d argue the challenges we face are often more similar than they are different.

With that in mind, maybe we should look at what can we learn from the OOP community’s best practices around class design.

What can OOP teach us about testing & design?
---------------------------------------------

### 1\. Test public interfaces

This principle is easier to enforce in OOP languages because the interaction between different units of code is more rigid. In Elixir, we have functions on modules and message passing; there’s _no_ protection available on the latter, and one might wish for better protections on the former. (There have been proposals for the concept of a private module, for instance, whose “public” functions would still only be available to certain parts of the system.)

#### 1.1 Test public functions

If you’re tempted to test a private function, ask yourself why. Changing a `defp` to `def` is fine while you’re doing TDD, but before you commit, you should make sure you have a higher-level test in place so that you can make the function once again private. There are a couple advantages to keeping these private:

1.  A smaller public interface makes a module easier to consume for your peers (or yourself in 3 months).
2.  Testing of (conceptually) private interfaces—implementation details!—couples your tests more tightly to this particular implementation; when the code changes, the tests have to change, to no real benefit. In contrast, if you’re testing at the feature level, your tests are more likely to fail only when the system’s behavior is wrong from a user’s point of view.

Thus, you shouldn’t be afraid to delete tests that are covered elsewhere.

#### 1.2 Probably don’t test message passing

We want to test public, observable behavior of the system.

That means your tests should be instrumenting public _functions_, not doing message passing, because in most cases, the specific structure of the messages that get passed are a (private) detail of the implementation.

Why does this matter? There’s a trap I see intermediate Elixir developers fall into. It goes something like: “We can save some boilerplate by interacting with our GenServer via `GenServer.call/3` or `cast/3`.” In even more extreme cases, they might skip the GenServer calls entirely—let’s just send a `:"$gen_call"` or `:"$gen_cast"` message!

Worse still, your tests might assert that certain `:"$gen_call"`/`:"$gen_cast"` messages were sent _back_.

As The Kids say, “this ain’t it.” Not only does it obscure your intent—anyone wanting to use your module has to dig through the messages it listens for or sends—but it tightly couples your tests and potentially production components to the implementation of your GenServer.

That said, there are some cases where message passing really _is_ your public interface (e.g., when you’re running distributed Erlang), but those cases tend to be exceedingly rare in the scope of a whole codebase.

### 2 Keep separate concerns separate

[Tony van Eerd](https://twitter.com/tvaneerd), a luminary in the C++ community, says that all advice for improving maintainability boils down to simply [“Keep your stuff separate”](https://adspthepodcast.com/2021/05/07/Episode-24.html) (48:20). (This insight also available [in tweet form](https://twitter.com/tvaneerd/status/1265268698698448898). 😉)

I really like this formulation of that idea, but the idea is not unique to Tony—for instance, [Dave Thomas](https://twitter.com/pragdave)’s [definition of good design](https://www.youtube.com/watch?v=6U7cLUygMeI&t=5m) gets at the same thing: “Well designed code is easier to change than badly designed code.” (What makes it easier to change? For starters, being less entangled with the rest of the system!)

What’s this look like in practice for GenServers?

When you’re designing a system, you will constantly be faced with decisions about where to draw lines between separate components.

Some decisions will be clear (at least if you can _formulate_ the question of where to draw the line). For instance, should we have separate modules for processing versus merely downloading the data we rely on from a third-party API, or should we encapsulate both functions in a single interface? Phrased that way, it seems pretty straightforward those are separate functions, and the data processor can accept the downloader as a dependency.

Other decisions will be murkier. Keeping with the theme of deciding a module’s responsibilities[¹](#footnotes), should we have separate modules for transforming a third-party API response into our domain types, or should that be bundled up with the GenServer that maintains the latest response state? It’s definitely possible to split responsibilities to _too_ fine a level.

This is perhaps the core question of designing software architecture, at least “in the small” (i.e., at the level of individual modules or small clusters of modules), and there’s no substitute for experience when it comes to developing an intuition around this stuff. With that said, in my experience it’s very rare to see folks who go overboard separating concerns, and very common to see modules that could be broken up further; if you have to pick a direction to adjust your intuitions around this sort of stuff, I’d recommend aiming for _greater_ separation than you’d otherwise be inclined to.

With that said, there’s one concrete suggestion I can make in this area that’s served me well every time I’ve done it:

#### 2.1 Separate the statefulness of your GenServer from the underlying data transformations

Perhaps the single most impactful (and often-quoted) idea I’ve been exposed to in the realm architecture is [Gary Bernhardt](https://twitter.com/garybernhardt)’s idea of the [separation between the “functional core” and “imperative shell”](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell) of your system.

That screencast is a mere 13 minutes long (you should absolutely watch it!), but the gist is that all systems have state (you can’t avoid this!), and that state makes things hard to test. What you _can_ do, though, is separate out the pure (functional) data transformations and test those in isolation. Taken to its logical conclusion, your imperative, stateful layer becomes a thin shell around the bulk of your (pure) business logic.

At the level of a single GenServer, my preferred implementation of this idea is to separate out the state-_maintaining_ portion from the state _manipulations_. Generally that means there’s an “implementation” module that provides and operates on a state struct, with the server implementation (`handle_call/3`, `handle_cast/3`, etc.) calling into the “Impl” module to do its transformations and queries.

Here’s a concrete example. It’s significantly more complicated than your typical “slideware” example, but I felt like it had to be in order to meaningfully illustrate the separation of concerns here.

Suppose your app needs to track logged in users associated with a region. (Maybe you’re [making a multiplayer server for your flight simulator](https://elixir-lang.org/blog/2021/07/29/bootstraping-a-multiplayer-server-with-elixir-at-x-plane/) or something. 😉) When a user travels between regions, they should be removed from whatever region they were previously associated with (if any) and placed in the new region. Likewise, if you haven’t heard from a user in awhile, you should assume their connection was interrupted or they ended their session.

This isn’t particularly tricky functionality to implement, but it’s juuuuuuuust complicated enough to be a pain.

When thinking about the design, I see four bits of functionality:

1.  There’s a data structure here that maps users to regions (or maybe regions to users?)
2.  There’s a “heartbeat” tracker that notes the last time we heard from a user
3.  There’s a timer that runs the “clean up users with expired heartbeats” call periodically
4.  There’s the actual stateful storage of the data (such that elsewhere in the app, we can call into the one process that’s storing this state for the whole app)

Now, if you take the “single responsibility principle[¹](#footnotes)” seriously, you might say to yourself “four pieces of functionality? I guess we have four modules here!” I am… not enamored with that way of approaching architecture. There’s a tight coupling between the first two items in that list, and also between the last two; you split those up at your own peril, because suddenly some _other_ piece of code is going to need to handle the orchestration _between_ the tightly coupled bits.

Thus, the division that makes sense to me is two modules:

*   one “Impl” module ([`UserCounter.Impl`](https://github.com/s3cur3/genserver_architecture_example/blob/main/lib/gen_server_example/user_counter_impl.ex)) that provides a struct, plus operations for:
    *   associating users with regions
    *   heartbeat tracking
    *   removing users with expired heartbeats
*   the actual GenServer module ([`UserCounter`](https://github.com/s3cur3/genserver_architecture_example/blob/main/lib/gen_server_example/user_counter.ex)) that acts as the stateful store of our struct and is responsible for periodically running the heartbeat expiration

In this setup, `UserCounter` is quite a thin wrapper around the struct—most of its body is boilerplate for exposing functions on the `Impl`, although the utility function [`GenImpl.apply_call/3`](https://github.com/s3cur3/genserver_architecture_example/blob/main/lib/gen_impl.ex) does help substantially cut down on that boilerplate. (I’d really like to one day have a macro that says “my GenServer wraps _all_ the public functions of my Impl module in the usual way.”)

The upshot is that [the test for the Impl](https://github.com/s3cur3/genserver_architecture_example/blob/main/test/user_counter_impl_test.exs) is comprehensive and thorough, but [the `UserCounter` test](https://github.com/s3cur3/genserver_architecture_example/blob/main/test/user_counter_test.exs) only really needs to test that the wire-up happened correctly, and that any GenServer-specific functionality (in this case, the periodic running of the heartbeat expiration call) works correctly.

What I like about this pattern is that it groups the business logic together much more tightly than you get from putting it inline in the GenServer. (For more on this complaint, see Dave Thomas’s EMPEX 2018 keynote.) How often have you gone to work on a GenServer and had to trace through the messages it sends to itself, then wade through the `handle_call/3` to separate out the parts that are GenServer boilerplate versus the actual data transformations? The Impl module has much higher cohesiveness, and the GenServer module… well, it still ends up with a lot of boilerplate, but at least the regularity of it makes it easier to take in at a glance.

Without this pattern, you frequently end up with _some_ functions in the GenServer that operate on a GenServer process, and others that operate on its state… and God help you as a maintainer when it isn’t obvious which is which!

**Post-publication edit**: This has been a sticking point for a lot of readers. I’ve gone back and forth over time between making the data structure the top-level thing (e.g., `UserCounter` is the struct, `UserCounter.Server` is the GenServer) versus the layout described above, where the GenServer is top-level. Based on reader feedback, it seems like the former is pretty strongly preferred in the Elixir community.

### 3 Dependency inversion, dependency injection, and inversion of control

Dependency inversion is an idea that comes from the Java design patterns community. The idea here is to not have module A depend on module B directly, but instead to depend on a shared, abstract interface which B _happens_ to implement[²](#footnotes). Taken to the extreme, this is a great way to write a whole lot of code to for very little actual value. (This is how you quickly get to writing an `AbstractWidgetSingletonFactoryInterface`.)

Dependency inversion looks a little different in Elixir, because the “unit” of dependency can be as small as a single function or as big as a protocol. At the small end of that spectrum it looks more like what we call dependency _injection_, but the lines get blurred quickly; there’s less value in classification than in understanding the problems we’re solving here. There’s one big takeaway from the dependency inversion stuff that I find useful: if you can expose a small public API for your functionality (whether that’s as a module, a behavior, a protocol, or even just a single anonymous function), _and_ that \[thing\] gets passed around to all the modules that use it, it becomes much easier to replace that thing.

I say _replace_ here intentionally, not “mock” or “stub,” because my experience is that mocking and stubbing remove a significant portion of the value from a test. Take the classic example of testing a module that depends on a third-party API call; you don’t want to make HTTP requests to a dependency you can’t control during tests, so you need to find some way to test around it. A few common approaches include:

*   Use a mocking framework. In my opinion, this couples your test far too tightly to the implementation details of your system. If the implementation changes, you’ll almost certainly have to go dig into the internals and figure out how your “expectations” around the mocked calls have changed. The test becomes less about testing the behavior of your system and more about duplicating the implementation.
*   Use some global configuration (e.g., `Application.put_env/4` in your test, then `Application.get_env/3` deep in the call stack of your test). Not only is this similarly brittle, but it’s easy to overlook when you’re writing the test and easy to fail to put the global config back into the correct state for the _rest_ of your tests. I prefer to avoid this kind of spooky action-at-a-distance whenever possible!
*   Explicitly pass the dependency you need to the module under test (either when you initialize your GenServer/struct or to the particular function call that needs it). As Chris Keathley [pointed out on a recent episode of Beam Radio](https://www.beamrad.io/18), the callsite knows the most about what it needs—why not let it directly, explicitly make decisions about key aspects of your API’s behavior?

This third option clearly strikes me as the best, but it can take a few different forms:

1.  Pass a behavior- or protocol-implementing module (or, so help me, just pass a module and treat it _as if_ it implements a behavior… I’m not recommending it, but I’ve done it, and it’s mostly fine). In the classic call-a-third-party-JSON-API example, this means you’d have an interface for requesting a certain path from the third-party; in tests you’d pass an implementation of that interface that returns hard-coded data, and in production it would actually make an HTTP request.
2.  Pass a function. This is the most minimal version, and probably most suited to the “dependency injection on a per-function call basis” version (rather than to the GenServer or struct at construction), but your needs may vary. In the third-party API example, this means you provide a single function that takes an endpoint and returns some data.

Back to my point about supporting “replacing” the dependency, rather than just providing the real versus test fake version: The advantage to doing it this way over a mock is that you can _genuinely_ change the production implementation without rewriting the systems that depend on it.

Imagine if you needed to add caching to your third-party API calls. If those calls are “raw” HTTP requests buried at the deepest level of your stack, you’re going to have to go change every single callsite to use the new caching version. But, if you’d structured that dependency explicitly from the start, you would find the topmost level where the HTTP interface gets passed in, swap it for the caching version, and you’re done.

(For that reason, I consider the fact that this pattern is massively more testable to be just icing on the cake.)

Feedback welcome
----------------

I generally try to avoid being controversial—I just don’t have the energy to argue with someone who is [wrong on the internet](https://xkcd.com/386/)—but part of the reason I wanted to air these opinions was to hear from people with conflicting experience. These ideas are going to be the basis of [my upcoming ElixirConf 2021 talk](https://www.elixirconf.com), and I’d love to improve that talk before it’s delivered.

You’re welcome to drop a comment below or reach out to me on Twitter [@TylerAYoung](https://twitter.com/TylerAYoung).

Footnotes
---------

¹ I do think of it as _plural_ responsibilities. The so-called [single responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle) sounds nice, but I find it fundamentally un-actionable at best, and misleading at worst. It can either lead you down the path of dividing things to an absurdly, unmaintainably fine degree, or if you squint hard enough, you can convince yourself that, say, “handling our entire client API” is a “single” responsibility.

² The keen observer will note that the idea of _merely_ depending on an interface is in direct opposition to the observed reality of Hyrum’s Law: given a sufficient number of users of an API, all observable behavior of your system will be depended on by somebody. And of course, there’s an [XKCD that illustrates this](https://xkcd.com/1172/).
