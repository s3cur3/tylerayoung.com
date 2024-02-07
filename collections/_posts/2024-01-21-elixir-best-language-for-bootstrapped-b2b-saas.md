---
title: "Why Elixir Is the Best Language for Building a Bootstrapped, B2B SaaS in 2024"
layout: post
authors: ['tyler']
categories: ["Entrepreneurship", "Build in Public"]
excerpt_separator: <!--more-->
image: /assets/images/elixir-one-person-stack-presentation.jpg
canonical_url: https://www.sleepeasy.app/2024/01/21/elixir-best-language-for-bootstrapped-b2b-saas/
---
<div class="w-full sm:float-right sm:w-72 sm:ml-8 mb-2 text-sm md:-mr-4 lg:-mr-24 xl:-mr-72 hidden-on-blog-listing">
<h3 class="!mt-0" style="margin-top: 0 !important">Contents</h3>
<ol>
    <li><a href="#who-cares-about-one-person-frameworks">Who cares about one-person frameworks?</a></li>
    <li><a href="#how-elixir-collapses-a-web-apps-tech-stack">How Elixir collapses a web app’s tech stack</a>
        <ol>
            <li><a href="#removing-layers-of-the-stack">Removing layers of the stack</a></li>
            <li><a href="#building-more-of-the-stack-into-the-platform-itself">Building more of the stack into the platform itself</a></li>
            <li><a href="#building-more-of-the-stack-using-tools-you-already-know">Building more of the stack using tools you already know</a></li>
            <li><a href="#whats-it-add-up-to">What's it add up to?</a></li>
        </ol>
    </li>
    <li><a href="#a-few-other-accelerators-for-saas-startups">A few other accelerators for SaaS startups</a>
        <ol>
            <li><a href="#buying-a-200-hour-head-start">Buying a 200 hour head start</a></li>
            <li><a href="#consuming-openapi-specifications-with-grace">Consuming OpenAPI specifications with grace</a></li>
            <li><a href="#maintainability-over-time">Maintainability over time</a></li>
            <li><a href="#wrapping-up">Wrapping up</a></li>
        </ol>
    </li>
    <li><a href="#appendix-breakdown-of-elixirs-answer-for-common-web-dev-requirements">Appendix: Breakdown of Elixir's answer for common web dev requirements</a></li>
    <li><a href="#footnotes">Footnotes</a></li>
</ol>
</div>

\[This article is the companion to my [presentation](https://codebeamamerica.com/talks/elixir-is-the-one-person-stack/) for CodeBEAM America 2024, Elixir is the One-Person Stack for Building a Software Startup. You can [download the slides as a PDF](/assets/files/elixir-one-person-stack-software-startup-codebeam.pdf) or [view them in Google Slides](https://docs.google.com/presentation/d/11aRQTWI-Jqenmqe66rMuWWcy_VhMHN5x-dzm7sAW7ac/edit#slide=id.g2b1888d014f_0_27).\]

I'd like to share why I chose Elixir as the programming language (and really, as we'll discuss, the full stack) for [SleepEasy](https://www.sleepeasy.app), the website monitoring SaaS I'm building. I'm going to do my best to focus on the objective features of the language which make it particularly suitable for a small, nimble team starting a software business.

Because SleepEasy is B2B software, a web app is absolutely required. At some point in the distant future, a mobile app may be too, but I expect to get by without mobile for a long time. Even if I do one day need a mobile app, a simple wrapper around a web view will probably suffice.

The fact that I'm bootstrapping this company (that is, self-funding to start and growing it solely from the business's own profits) sets one other major requirement: the app needs to be able to be built and maintained by a team of one, at least for the first few years or the first $10k+ in monthly revenue.

<!--more-->

## Who cares about one-person frameworks?

<!--[Late last year](https://world.hey.com/dhh/the-one-person-framework-711e6318), DHH called Rails 7 “the one person framework,” and since then I’ve seen a lot of discussion online about this idea. It's kind of the dream, right? Whatever software you're building, being able to fit the whole in one person’s head is a major advantage. To have one _real human_ be able to build and maintain it, full stack, is really important. -->

<!--Of course, I have to specify a _real_ human because modern web stacks have become unspeakably complex.-->

Look at any job posting for a full stack developer and consider just how many things they're expected to have expertise in. Every employer is trying to find a unicorn who knows:

<figure style="float: right; margin-left: 2rem; margin-top: 0; margin-bottom: 0.5rem; width: 280px;">
<img style="margin: 0; margin-bottom: 0.5rem" width="280px" src="/assets/images/tom-and-jerry-orchestra.gif" alt="Jerry, of Tom and Jerry, playing a dozen orchestra instruments at once" />
<p class="text-gray-600 text-sm">Pictured: A full-stack dev orchestrating two dozen tools to build a single web app</p>
</figure>

- HTML
- CSS
- Tailwind
- A frontend language (JavaScript/TypeScript)
- A backend language (Ruby, Python, Go, etc.)
- A frontend framework (React, Vue, etc.) 
- A frontend state management framework (Redux, Jotai, Vuex, etc.)
- A backend framework
- REST
- GraphQL
- A SQL database
- A NoSQL database
- A background job system
- An in-memory cache like Redis
- A service crash recovery system (PM2, Upstart, etc.)
- A message queue (RabbitMQ, Redis, etc.)
- A web server like Nginx
- Docker
- Kubernetes
- A cloud platform (AWS, GCP, Azure)
- Serverless
- Microservices
- Scaling services

🥴

It's too much! It's simply not reasonable to expect one person to be able to do it all. And that's doubly true for someone starting a solo software company, where you're _also_ responsible for customer development, marketing, sales, and all the other parts of the business.

All this leads to one inescapable conclusion:

> **We have to collapse the stack!**

We need to _dramatically_ cut down on the number of different technologies you need to learn to build a best-in-class web app. That's where Elixir (and specifically Elixir plus the Phoenix web framework) comes in.


## How Elixir collapses a web app's tech stack

There are three big ways Elixir helps simplify web application development.

1. Removing layers of the stack entirely
2. Building more of the stack into either the language, the standard library, or Erlang's BEAM + OTP platform<a href="#footnote-1" id="footnote-1-source">¹</a>
3. Building more of the stack in tools you already know

Let me explain...

### Removing layers of the stack

[Phoenix LiveView](https://github.com/phoenixframework/phoenix_live_view) has gotten a ton of positive attention<a href="#footnote-2-source" id="footnote-2">²</a>, and for good reason. The pitch is that you can create rich, interactive client-side experiences (comparable to a SPA framework like React or Vue) while writing _just_ "backend" code. By building on Phoenix's excellent WebSocket support, LiveView provides:

- SPA-like page transitions (i.e., replacing just the parts of the page that change, without a full page reload),
- real-time "reactive" updates of the client-side view as state changes on the backend, and
- server interactivity without ever needing to construct an API or write JavaScript.

And all of this comes more or less for free. Seamless, sub-50 ms page transitions? 0 lines of code.  Triggering backend events from the from the frontend? 3-4 lines of code. Subscribing the frontend to progress updates on some backend job? 4-6 lines of code.

There are caveats, of course. LiveView has a substantial learning curve on its own, and I wouldn't advise trying to build something that's fundamentally un-document like. (There's a reason we built [Felt](https://felt.com) as a SPA talking over WebSockets to our Phoenix backend.) But again, if you're building a B2B SaaS, 95% of the time the product boils down to an admin dashboard, a CRUD app, or an ecommerce platform... not the next Figma.

Has LiveView replaced 100% of JavaScript for me? Of course not. But thankfully it ships with support for "hooks," such that you can delegate bits of functionality to client-side JavaScript (including SPA frameworks if that's your thing) while keeping the rest of your app in Elixir. After four months of development on SleepEasy's MVP, I have a total of 16 lines of JavaScript in the project.

On top of the concrete benefits of just needing to master fewer technologies, it’s hard to overstate how nice it is to keep your head in one place (the backend, in this case) for the vast majority of the development. Having to think about the interplay between client-side JavaScript, a REST API, and the backend was like wearing a weighted vest since birth. It wasn’t something I thought about as being a drag, but once it was removed, it felt like I was walking on air.

### Building more of the stack into the platform itself

Elixir has similar stack-shrinking benefits beyond LiveView too. The BEAM and OTP provides built-in support for a lot of concurrency and fault tolerance tooling that has to be bolted on in other ecosystems.

- Elixir's fault tolerance primitives (the [process isolation and supervision tree model](https://hexdocs.pm/elixir/main/supervisor-and-application.html)) remove the need for crash recovery at the whole-service level
- Erlang's ETS tables offer the in-memory caching functionality most apps need from Redis, but without needing to spin up a separate service (and dealing with all the things that can go wrong in a distributed system like that)
- Phoenix PubSub provides an in-memory message queue that can replace something like RabbitMQ
- The platform's thoughtful design for concurrency prevents any single process from starving the rest of the system for resources, so you can have thousands of concurrent requests on a single machine without worrying about them conflicting with one another.

### Building more of the stack using tools you already know

Finally, Elixir simplifies applications by having an ecosystem built on tooling you already know. That sounds a little weird, but consider the job queueing system. There are two main ways Elixir handles background jobs:

- One is by using the BEAM's built-in, effortless concurrency model (usually via [Task](https://hexdocs.pm/elixir/Task.html) or, in a roundabout way, via [GenServer](https://hexdocs.pm/elixir/main/genservers.html))—this is suitable for any ephemeral tasks that don't need to be robust against server reboots.
- The other is using a library called [Oban](https://github.com/sorentwo/oban), which is comparable to Ruby's Sidekiq.

Oban runs on top of Postgres (or SQLite, if that's your thing), unlike Sidekiq and similar systems that are backed by Redis. That reduces the number of technologies you need to learn (and deploy, and manage!) by one, since presumably you already need to know your SQL database.

Elixir has also simplified my deployment model this way. Because of that fantastic concurrency model I've been going on about, Elixir scales extremely well as you increase the number of CPU cores and amount of RAM on the system. Vertically scaling like this is way, *way* easier than scaling out to more machines running your application—or worse, microservices!—because you avoid introducing distributed systems problems that serve as a drag on all future development. It takes zero lines of code change and zero additional testing to pay a little more for a bigger machine... that's not something you can say about scaling out a distributed system! (As an added benefit, it's super cheap to deploy a single monolith talking to a single database!)

The final area where the Elixir stack builds more of the stack in tools that you already know is around testing. While ExUnit is _amazing_ and I could sing the praises of its readability for days (how many other ecosystems have the entire community using the testing tool that ships with the language?), the fact that there's _some_ unit testing framework in Elixir isn't that remarkable. What's amazing is the testing story around LiveView.

Remember how LiveView lets you build frontend interactivity from the backend? It also lets you write _tests_ of your frontend interactions in ExUnit, rather than needing browser automation which is inherently both slower and flakier. You can make assertions like "when I fill in these form fields and click this button, I should be redirected to a page with the title of \_\_\_\_\_\_\_." The cost of writing these integration tests—in terms of runtime, development time, cognitive load, and general pain-in-the-ass factor—is more or less the same as if I were testing a pure function in my business logic, and I find myself writing _way_ more tests than I ever did for a React SPA. If I have to manually test something more than once, you can bet it's going to become an integration test.

### What's it add up to?

Let's go back to the original list of technologies a full-stack dev is expected to know and see how many of them we can replace or remove with the Elixir stack I've described here. By my count, we go from 23 things a web app can reasonably be expected to need down to 8 (counting anything built into Elixir as one technology to learn, and anything built into Phoenix as another):

1. Elixir (including supervision trees for fault tolerance, concurrency primitives like `Task`, and ETS for caching)
2. Phoenix (including LiveView and PubSub)
3. Oban for robust background jobs
4. Postgres
5. The PaaS of your choice (I prefer to self-host with Dokku, a Heroku-like self-hosted PaaS; others prefer Render, Fly.io, or Gigalixir)
6. HTML
7. CSS
8. Tailwind

That's not bad, especially considering you're probably coming into Elixir with maybe half those skills.

For a complete breakdown of the Elixir ecosystem's answer to each of the original list of things a full-stack dev was expected to juggle, see [the appendix below](#appendix-breakdown-of-elixirs-answer-for-common-web-dev-requirements).

## A few other accelerators for SaaS startups

Using the stack I've laid out above, you could build 95% of B2B SaaS apps, and you could do it faster and more reliably than any other ecosystem I've seen. That said, there are a few more areas of the Elixir ecosystem that make it a great fit for bootstrapped startups, and I'd be remiss not to highlight them.

### Buying a 200 hour head start

The first is [the Petal Pro framework](https://docs.petal.build/petal-pro-documentation/). "Petal" there is a reference to the PETAL stack: Phoenix, Elixir, Tailwind, Alpine JS, and LiveView. (It's a nice acronym, but since LiveView introduced `LiveView.JS`  back in 2022, you can handle purely client-side interactions like toggling visibility of a modal without the need for Alpine at all.)

Petal Pro gives you a head start on implementing an absolute _ton_ of functionality that will either be an absolute requirement for every SaaS app, or are *extremely* nice to have for monitoring, debugging, and providing support. I've built most of these from scratch in the past, and they're all totally doable, but they take time. Being able to spend $300 to not have to think about them again is an absolute _steal_.

A few of the biggest time-savers for me:

- Stripe integration for doing subscription billing
- Organizations for users to group into (including sending and accepting org invitations)
- Admin dashboards (and a toolkit for building your own admin dashboards that lets me churn out new dashboard views in an hour which would have taken me *days* before)
- User impersonation, so that when a user reports a problem, I can log in and see exactly what they see
- A nicely designed LiveView component library, complete with page layouts, menus, and dark mode support for everything

### Consuming OpenAPI specifications with grace

Next, there's always the concern around ecosystem size, and it's true, Elixir's ecosystem is way smaller than NPM or PyPI. Now, in practice, I've found the holes in the package ecosystem to not be too bad. If you just need a few REST endpoints from a third-party service, it's not hard to write those integration. (I cut my teeth in C++, though, where writing your own implementation for dependencies was not just encouraged, but often the easiest path!) But, if you need deep integration with a huge third party API, that might be a non-starter.

That's where AJ Foster's [`open-api-generator`](https://github.com/aj-foster/open-api-generator) comes in. Unlike most OpenAPI generators, it offers a way to do deep customization of the auto-generated code to produce an ergonomic Elixir API. Rather than consuming the OpenAPI spec for your third party and vomiting it out wholesale (leading to a crummy API that a human would never produce by hand), the generator gives you ways to:

- Rename components of the API
- Group schemas into module namespaces
- Merge multiple, nearly-synonymous data structures into one
- ...and much more

You can see compare [AJ's GitHub API wrapper](https://github.com/aj-foster/open-api-github) to what you get by default when you spit out the GitHub OpenAPI, and it's night and day... and at a scale that an unpaid volunteer could never match if they tried to wrap the GitHub API by hand.

AJ gave a great talk at last year's ElixirConf showing off the power of this stuff:

<iframe width="560" height="315" src="https://www.youtube.com/embed/XSwxNgza7hE?si=p0xQt9Iu0LawU9_7" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen style="margin-inline: auto;"></iframe>


### Maintainability over time

The last thing I'd like to mention is how very little churn there is in the Elixir ecosystem. In stark contrast to other stacks I've worked in, where taking even a "patch" update to a framework can require even experts to put in _hours_ of frustrating debugging (as [Gary Bernhardt recently bemoaned](https://twitter.com/garybernhardt/status/1748486057920963041)), taking an update to Elixir or Phoenix is not much of an issue. If you're like me and treat warnings as errors, you'll frequently hit a few deprecations and the like, but those are almost always an easy fix. And that's reflected in a recent pair of polls I ran<a href="#footnote-3" id="footnote-3-source">³</a>. The overwhelming majority of users are on versions of Elixir and Phoenix released within the last year or so, and less than 5% are on versions more than 3 years old.

[![Elixir versions used in production; 82.1% of poll respondents are using Elixir 1.15 or newer; 15.5% are using 1.13 or 1.14; 0.5% are using 1.11 or 1.12; 1.9% are using 1.10 or earlier](/assets/images/elixir-versions.png)](https://x.com/TylerAYoung/status/1748423976609554728)

[![Phoenix versions used in production; 78.5% of poll respondents are using Phoenix 1.7; 16.8% are using 1.6; 1.9% are using 1.5; 2.8% are using 1.4 or earlier](/assets/images/phoenix-versions.png)](https://x.com/TylerAYoung/status/1748689778650849630)

Elixir and Phoenix value stability, so it's generally easy to get access to new features without a bunch of hassle.

## Wrapping up

I'm not qualified to say Elixir is the right language choice for all apps everywhere. I've never worked in a big corporation, and my experience with Elixir has been largely focused on web and networking. I *do* feel comfortable evaluating it for the project I'm working on now, though, and for the needs of a one-person development team building a B2B SaaS, I don't see any other stack that offers both the speed of getting started and the ability to grow in whatever direction your business takes you.

I'd love to hear any feedback you have—you can reach out to me on [Twitter](https://twitter.com/TylerAYoung), [Mastodon](https://fosstodon.org/@tylerayoung), or email (my first name at this domain).

## Appendix: Breakdown of Elixir's answer for common web dev requirements

<style>
    @media (min-width: 769px) {
        table {
            width: 85vw !important;
            margin-inline: calc(50% - 50vw);
        }
    }
    @media (min-width: 1200px) {
        table {
            width: 70vw !important;
            margin-inline: calc(33% - 33vw);
        }
    }
</style>

| Technology                            | The typical way                              | The Elixir way I'm advocating                                                                                                                                                                  |
| ------------------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HTML                                  | Gotta learn it                               | Still gotta learn it                                                                                                                                                                           |
| CSS                                   | Gotta learn it                               | Sorry, still gotta learn it                                                                                                                                                                    |
| Tailwind                              | Optional, but nice                           | Optional, but nice                                                                                                                                                                             |
| A frontend language                   | JavaScript/TypeScript                        | Phoenix LiveView                                                                                                                                                                               |
| A backend language                    | Ruby, Python, Go                             | Elixir                                                                                                                                                                                         |
| A frontend framework                  | React, Vue                                   | Phoenix LiveView                                                                                                                                                                               |
| A frontend state management framework | Redux, Jotai, Vuex                           | N/A with LiveView                                                                                                                                                                              |
| A backend framework                   | Rails, Next.js, Django                       | Phoenix LiveView                                                                                                                                                                               |
| REST                                  | Needed for client-server communication       | Unnecessary with LiveView <br>(Phoenix if you need it for product reasons)                                                                                                                     |
| GraphQL                               | Maybe needed for client-server communication | Unnecessary with LiveView <br>(Absinthe if you need it for product reasons)                                                                                                                    |
| A SQL database                        | Postgres, MySQL, SQLite                      | Postgres                                                                                                                                                                                       |
| A NoSQL database                      | Mongo, CouchDB                               | Postgres JSONB columns or in-memory caching with ETS                                                                                                                                           |
| A background job system               | Sidekiq, Celery, BullMQ                      | Built-in `Task` or Oban library                                                                                                                                                                |
| An in-memory cache                    | Redis                                        | ETS, or a thin wrapper around ETS like Cachex                                                                                                                                                  |
| A service crash recovery system       | PM2, Upstart                                 | Built-in fault recovery via Supervisor trees                                                                                                                                                   |
| A message queue                       | RabbitMQ, Redis                              | Phoenix PubSub                                                                                                                                                                                 |
| A web server                          | Nginx, Apache, Gunicorn                      | Phoenix                                                                                                                                                                                        |
| Containerization                      | Docker                                       | PaaS like Render, Fly.io, Gigalixir, or Dokku that abstracts over containers (or bare binary release deployments)                                                                              |
| Container orchestration               | Kubernetes                                   | PaaS or bare metal deployments                                                                                                                                                                 |
| A cloud platform                      | AWS, GCP, Azure                              | PaaS or bare metal deployments                                                                                                                                                                 |
| Serverless                            | AWS Lambda                                   | Vertically-scaled monolith <br> [FLAME](https://fly.io/blog/rethinking-serverless-with-flame/) if you really need serverless-like scaling or to seamlessly run functions on different hardware |
| Microservices                         | Pain                                         | Monolith with many cores <br> [Boundary](https://hexdocs.pm/boundary/readme.html) if you need to ensure separation of concerns between teams                                                   |
| Scaling services                      | Horizontal                                   | Vertical, only horizontal if you really need redundancy or multi-region deployments                                                                                                            |

## Footnotes


<a href="#footnote-1-source" id="footnote-1">¹</a> "The BEAM" is the name of the Erlang virtual machine on which Elixir is built, and OTP (the "Open Telecom Platform") is the set of core Erlang abstractions and libraries for things like process isolation, networking, and distributed computing.

<a href="#footnote-2-source" id="footnote-2">²</a> Phoenix has been ranked the "most loved web framework" two years running in StackOverflow's developer survey ([2022](https://survey.stackoverflow.co/2022/#section-most-loved-dreaded-and-wanted-web-frameworks-and-technologies), [2023](https://survey.stackoverflow.co/2023/#section-admired-and-desired-web-frameworks-and-technologies)).

<a href="#footnote-3-source" id="footnote-3">³</a> Admittedly unscientific, but with 200+ respondents to the Elixir poll and 100+ to the Phoenix version, it seems like a reasonable snapshot of the ecosystem.
