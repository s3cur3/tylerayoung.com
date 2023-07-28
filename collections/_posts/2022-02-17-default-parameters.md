---
title: "TIL about default parameters in Elixir"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
image: "/assets/images/default-params.png"
excerpt_full: true
---

TIL default parameters in Elixir that are *function calls* get re-evaluated every time you call the function.

This is how you'd hope this sort of thing works, but I think years of being burned by Python made me mistrust it.

Somehow I missed this my first time through the *outstanding* Testing Elixir by Andrea Leopardi and Jeffrey Matthias. Every time I open that book up I get something new and valuable. 

This is a godsend for testability!
