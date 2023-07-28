---
title: "TIL you can have type predicates in TypeScript"
layout: post
authors: ['tyler']
categories: ["Programming", "TypeScript", "Today I Learned (TIL)"]
image: /assets/images/type-predicate.png
excerpt_full: true
---

TIL in TypeScript you can have type predicates, where the return value of your function indicate the type of an argument. 

Instead of:
```ts
function isUserRole(role: string): boolean { }
```

...you can do:

```ts
function isUserRole(role: string): role is UserRole { }
```

That can be used to inform the type system of the value's type, letting you do this:

```ts
const validatedRole =
  isUserRole(roleInput) ?
    roleInput :
    undefined;
```

...and use `validatedRole` as an optional `UserRole` rather than just a string.


