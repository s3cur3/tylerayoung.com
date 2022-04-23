---
title: "Go ahead, call yourself a software engineer."
layout: post
authors: ['tyler']
categories: ["Programming"]
summary: Ian Bogost writes for The Atlantic that programmers should stop calling themselves engineers. It’s a good article, and he’s right about software devs cheapening the word “engineer” with their Wild West practices. This is a sentiment that’s been echoed for years.
---

[Post-publication edit: Hillel Wayne has an excellent [3-part series on this topic](https://www.hillelwayne.com/post/are-we-really-engineers/) that goes into depth I couldn't begin to myself.]

Ian Bogost writes for The Atlantic that [programmers should stop calling themselves engineers](https://www.theatlantic.com/technology/archive/2015/11/programmers-should-not-call-themselves-engineers/414271/). It’s a good article, and he’s right about software devs cheapening the word “engineer” with their Wild West practices. This is a sentiment that’s been echoed for years.

At the same time, though, there’s a worthwhile distinction to be drawn between “I banged this script out for one-time use and never intend to look at it again” and “I’ve put a lot of thought into the design of this, such that we should be able to maintain it for decades to come.”

Titus Winters of Google [described](https://www.youtube.com/watch?v=tISy7EJQPzI) software engineering as “programming integrated over time”—that is, programming that’s built to last.

In my own work, I run into this all the time. A “programmer” will say things like “it worked when I wrote it!”… but they didn’t:

*   write any tests for it (to make sure it continues to work in the future, or works on other machines)
*   think about any edge cases the code should handle
*   think about configuration or deployment
*   put any effort into making the code understandable by the person who will eventually have to maintain it
*   write any documentation on how it should be used, or even WTF it’s supposed to do

So people are entirely right that engineers who build bridges are operating at an entirely different level than software engineers in terms of reliability, planning, oversight, etc. But at the same time, people with a “software engineering” mentality are operating very differently from people with a “programming” mentality. Whatever we call that difference, it’s an important distinction.





