---
title: "Default Capacity & Growth Rate of C++ std::vector"
layout: post
authors: ['tyler']
categories: ["Programming", 'Cpp']
---

If you’re creating a lot of small vectors in C++, you should probably consider using [a small-size optimized vector](https://tylerayoung.com/2019/01/29/benchmarks-of-cache-friendly-data-structures-in-c/) instead. But, if you can’t do that for some reason, you might wonder if there is any win to be had by `reserve()`ing a small size in advance.

While the standard doesn’t _require_ vectors to have any particular initial `capacity()`, it seems like all the common implementations:

1.  Start with zero capacity
2.  Allocate a single element on the first `push_back()`/`emplace_back()`
3.  Grow to a pair of elements on the second push
4.  Grow at something vaguely resembling exponential on subsequent pushes

Here’s [a Compiler Explorer sample](https://godbolt.org/z/GnhGd1) to check this behavior on various compilers (adapted from [metamorphosis’ answer on StackOverflow](https://stackoverflow.com/a/36371057/1417451)):

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v;
    std::cout << "Initial capacity: " << v.capacity() << "\n";
    for(int i = 0; i != 10; ++i)
    {
        v.emplace_back(i); // use push_back() for pre-C++11 compilers, of course
        std::cout << "Size " << v.size() << " capacity: " << v.capacity() << "\n";
    }
    return 0;
}
```

GCC 5.1, GCC 10.2, Clang 6, and Clang 10.0.1 all grow as follows:

    Initial capacity: 0
    Size 1 capacity: 1
    Size 2 capacity: 2
    Size 3 capacity: 4
    Size 4 capacity: 4
    Size 5 capacity: 8
    Size 6 capacity: 8
    Size 7 capacity: 8
    Size 8 capacity: 8
    Size 9 capacity: 16
    Size 10 capacity: 16

While MSVC 2013 and 2019 both have a much looser interpretation of exponential capacity growth:

    Initial capacity: 0
    Size 1 capacity: 1
    Size 2 capacity: 2
    Size 3 capacity: 3
    Size 4 capacity: 4
    Size 5 capacity: 6
    Size 6 capacity: 6
    Size 7 capacity: 9
    Size 8 capacity: 9
    Size 9 capacity: 9
    Size 10 capacity: 13






