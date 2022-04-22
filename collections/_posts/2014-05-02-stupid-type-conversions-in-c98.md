---
title: Stupid Type Conversions in C++98
categories: ['Cpp']
authors: ['tyler']
layout: post
---

If you’re working on a C++98 project, you have my condolences.

There are a number of type conversions that newer versions of C++ make super easy, but which are not included in C++98.

Here’s my reference for these:

String to…
----------

### Int

You need the equivalent of `stoi()`. Here’s mine:

```cpp
static int stoi( std::string & s ) {
    int i;
    std::istringstream(s) >> i;
    return i;
}
```
    

### Double

Incredibly, C++98 has `atof(const char*)`. (You’ll just need to call `.c_str()` on your std::strings.)

Number (int or double) to String
--------------------------------

You need the equivalent of `std::to_string()`. Here’s mine:

```cpp
#include <sstream>

template<typename T>
std::string to_string(const T & value) {
    std::ostringstream oss;
    oss << value;
    return oss.str();
}
```
    


