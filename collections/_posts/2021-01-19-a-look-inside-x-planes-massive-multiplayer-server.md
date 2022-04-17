---
title: "A look inside X-Plane’s massive multiplayer server"
layout: post
authors: ['tyler']
categories: ["Programming", 'Game Development', "Elixir"]
---

This morning I published a blog post on the X-Plane Developer blog titled “[Have You Heard the Good News About Elixir?](https://developer.x-plane.com/2021/01/have-you-heard-the-good-news-about-elixir/).” It’s a look at the requirements and goals that drove me to choose Elixir for X-Plane’s [massive multiplayer](https://www.x-plane.com/2020/06/mobiles-massive-multiplayer-released/) game server, with a look at both the pros and cons of that choice.

This post (and the feature itself!) was a long time in coming. It’s cool to be able to talk about it publicly.

On a related note, we [open-sourced the Elixir implementation of the networking protocol](https://github.com/X-Plane/elixir-raknet) we use for MMO. The README there gives a pretty good overview of the architecture of our MMO server—long story short, we go to great lengths to minimize shared state.

This won’t be of use to many people, but I do hope it’s useful to the next poor soul who needs to build a server around RakNet. ☺️






