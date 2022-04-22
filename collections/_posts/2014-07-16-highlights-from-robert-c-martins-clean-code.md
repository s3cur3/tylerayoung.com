---
title: Highlights from Robert C. Martin’s Clean Code
categories: ['Programming']
authors: ['tyler']
layout: post
---

This page collects the things I found really insightful in Martin’s _Clean Code_. By “insightful,” I mean things I _didn’t_ already practice as a programmer with a couple years of experience. Thus, I’ve skipped over tips like “don’t be afraid of long variable names” in favor of things like “functions should act at one level of abstraction.”

Chapter 1: Introduction
-----------------------

> You know you are working on clean code when each routine you read turns out to be pretty much what you expected.

—Ward Cunningham

In other words, clean code reads as though it is obvious, simple, and compelling.

It’s the difference between watching a professional athlete (who makes the job look easy) and an amateur (who makes you appreciate how difficult the sport really is). Your code should give your readers the sense that the problem you’ve solved is unexpectedly easy (even when it isn’t).

Chapter 2: Variables
--------------------

### Add meaningful context

A variable’s name might be ambiguous if the context isn’t clear. Consider a variable called `state`; when used alone in a function, it’s not clear that it is part of an address. Consider either

*   wrapping these variables in a class (such that their context is _always_ clear), or
*   adding a prefix to the variable name so that the reader will at least be pointed to the larger structure (e.g., use `addrState` instead of `state`).

Chapter 3: Functions
--------------------

### Functions should act at one level of abstraction

It’s confusing when a function mixes multiple levels of abstraction—for instance, if it acts at a very high level of abstraction by calling `getHTML()` and a very low level of abstraction by manipulating low-level strings.

When this happens,

> Readers may not be able to tell whether a particular expression is an essential concept or an implementation detail.

Thus, you should hide steps at different levels of abstraction in different functions.

### Get rid of function parameters—especially low-level ones

The more arguments a function takes, the harder it is to use (it requires more mental energy) and test.

If an argument is at a different level of abstraction (e.g., `getUserData( StringBuffer htmlBuffer )`), it’s even worse—the user is forced to know an implementation detail in order to invoke the function!

