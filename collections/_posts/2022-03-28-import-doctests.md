---
title: "TIL: Elixir doctests support <code>import: true</code>"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
image: "/assets/images/doctests-import.png"
excerpt_full: true
---

TIL Elixir's doctests support an `import: true` flag so that you don't have to write out the full module name in your doctests.

Super useful! I can't tell you how many times my doctests have all started
`App.Context.Module.Submodule.func(...)`!
