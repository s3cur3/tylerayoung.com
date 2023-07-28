---
title: "TIL Postgres NULL comparisons rules can bite you on WHERE &hellip; NOT IN"
layout: post
authors: ['tyler']
categories: ["Programming", "Databases", "Today I Learned (TIL)"]
summary: Say you want to build a tool to handle translating a user interface. Whether in a native app or on the web, this seems like it should be pretty simple. You'll take each string in your UI, run it through a function that looks the string up in the user's preferred language, and you're done.
---

TIL in Postgres, if you do this:

```sql
SELECT * FROM things WHERE property NOT IN ('foo', 'bar', 'baz');
```

…it won't take rows where `property` was `NULL`. 

Surprising, but under the hood, it's trying to do an equality comparison, which never works on `NULL`.

[Further discussion on StackOverflow](https://stackoverflow.com/questions/17170856/using-the-not-in-operator-on-a-column-with-null-values)
