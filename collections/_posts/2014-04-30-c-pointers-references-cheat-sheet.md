---
title: C++ Pointers & References Cheat Sheet
categories: ['Cpp']
authors: ['tyler']
layout: post
---

My dirty little secret: I’ve spent too much time in fancy-pants languages like Java… and Python… and Ruby… and PHP… and Javascript… to remember what the `&` does all the time.

This is my cheat sheet. Hope it helps you too.

The Golden Rule
---------------

> If `&` is used to declare a variable, it makes that variable a reference.
> 
> Otherwise, `&` is used to take the address of a variable.

Pointers
--------

### Initialization

```cpp
int myInteger = 42;
int myIntPointer = &myInteger; // "address of"
cout << *myIntPointer; // prints 42
```

### As a parameter

```cpp
void foo( std::string * stringPtr ) {
    cout << *stringPtr; // prints "Aloha"
}
std::string s = "Aloha";
foo( &s ); // pass Aloha's address to the pointer
```

References
----------

### Pass-by-value

```cpp
void foo( int x ) {
    x = 42; // x is passed by value, so no change occurs outside this function
}
int x = 0;
foo(x);
// x == 0
```
    

### Pass-by-reference

```cpp
void fooMutate( int &x ) {
    x = 42; // passed by reference, so x changes outside this scope
}

int x = 0;
fooMutate(x);
// x == 42
```
    

### Declaring a reference (alias)

```cpp
int x = 0;
int y = &x; // y is now an alias of x
y = 42;
// x == 42
```



