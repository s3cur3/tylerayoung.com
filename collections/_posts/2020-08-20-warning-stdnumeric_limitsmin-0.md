---
title: "Warning: std::numeric_limits::min() > 0"
layout: post
authors: ['tyler']
categories: ["Programming", 'Apple']
---

Here’s a baffling design choice (which I’m reliably informed C++ inherited from older C `limits.h`).

`std::numeric_limits<int>::min()` is roughly -2 billion (assuming 32 bit ints, etc.).

But `std::numeric_limits<float>::min()` is smallest _positive_ floating point value… something like +0.00000000000000000000000000000000000001.

CppReference warns this is the case for all floating point types with denormalization (thus it applies to `double` and `long double` as well on vaguely x86-like platforms).

**What you’re probably looking for** (the _actual_ opposite of `std::numeric_limits<float>::max()`) **is `std::numeric_limits::lowest()`**.




