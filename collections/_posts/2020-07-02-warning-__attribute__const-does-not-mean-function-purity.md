---
title: "Warning: __attribute__((const)) does not mean function purity"
layout: post
authors: ['tyler']
categories: ["Programming", 'Cpp']
---

Clang and GCC (and others) support function attributes `__attribute__((const))` and `__attribute__((pure))`. These had always been explained to me thus:

*   `__attribute__((pure))`: The function may read, but doesn’t _modify_ any global state. \[This is true!\]
*   `__attribute__((const))`: The function neither reads nor modifies global state. \[**This is MASSIVELY OVERSIMPLIFIED** to the point of being a lie!\]

The **_gigantic_** asterisk on that simplified explanation of `__attribute__((const))` is that pointers and references—even const pointers and references!—are disallowed if the memory they reference can change between successive invocations of the function.

Consider:

```cpp
bool ends_with(const string & full, const string & suffix) __attribute__((const));
```

The implementation of this should certainly meet the requirements of neither reading nor writing global state… _unless_ you consider the memory referenced by your parameters to be global state!

But that’s exactly what GCC’s [docs](https://gcc.gnu.org/onlinedocs/gcc/Common-Function-Attributes.html#Common-Function-Attributes) on the attribute _do_ consider as global state:

> Note that a function that has pointer arguments and examines the data pointed to must _not_ be declared `const` if the pointed-to data might change between successive invocations of the function. In general, since a function cannot distinguish data that might change from data that cannot, const functions should never take pointer or, in C++, reference arguments.

(Note that Clang, despite having supported the attribute for years, has zero docs on it. Based on experience, though, it seems to have to the same restriction.)

The consequence of violating this requirement is that in the following code, `result2` _might_ wind up being true (if you get unlucky with the optimizer):

```cpp
string s = "foo";
bool result1 = ends_with(s, "oo"s); // True, always
s = "bar";
bool result2 = ends_with(s, "oo"s); // True, maybe!
```

That’s a somewhat contrived example, but it can be much more insidious when it’s a member variable you’re passing to your `__attribute__((const))` function. This can also bite you when you have _implicit_ conversions to const ref (e.g., when you’re passing a string literal to a function that converts it to a const string reference).

The worst part of all this—and the reason I’ve spent 2 full days over the course of the last few months debugging this—is that neither the compiler nor UBsan provide any warnings that this is happening.

If you squint really hard, you can kind of see what the compiler implementers were thinking—what you’re “really” passing here is just a pointer, which (again if you squint hard enough) is really just an integer… it’s _your_ fault, silly programmer, for turning looking at the (“global”) memory state corresponding to that int.

This is what people mean when they say C++ is a language of footguns. 🙁





