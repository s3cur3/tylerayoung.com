---
title: Notes from Andre Alexandrescu’s Modern C++ Design
categories: ["Programming", 'Cpp']
authors: ['tyler']
layout: post
---

Here are my highlights from Andre Alexandrescu’s _Modern C++ Design: Generic Programming and Design Patterns Applied_.

1.  Policy-Based Class Design
    *   **Overview**:  
        
        > Policy-based class design fosters assembling a class with complex behavoir out of many little classes (called _policies_), each of which takes care of only one behavioral or structural aspect.
        
    *   The challenge in any design is that you’re forced to choose a solution from a combinatorial solution space.
        *   Experienced designers are like experienced chess players: they can see many “moves” in the future and anticipate tradeoffs from a particular “correct” design choice.
    *   A “do-it-all” interface will fail because of the growing gap between _syntactically valid_ and _semantically valid_ uses of the library.
    *   Design-targeted libraries need to help user-crafted designs to enforce _their own_ constraints, rather than enforcing predefined constraints.
    *   Policy-based design combines the advantages of **multiple inheritance** and **templates**, while simultaneously avoiding their weaknesses.
        *   **Policy**: a class interface (or class template interface) with inner type definitions, member functions, and member variables that **emphasizes behavior rather than type**.
            *   Reminiscent of the Strategy design pattern
            *   Unique advantages: **static binding** and **type knowledge**
        *   Unlike a Java-style **interface**, a policy “interface” is loosely defined. They specify syntactic structures that should be valid rather than the exact functions that should be implemented.
            *   As a consequence, **different implementations of a policy may have different (or extended) behavior**.
                *   If you use a function not provided by all policy classes, your code will work just fine until you change concrete policies (as you would expect).
        *   Host classes assemble policies together into a single complex unit. (E.g., WidgetManager uses the Creator policy to define how widgets get created (p. 9).)
        *   By using template template parameters, you can avoid redundant (or easily inferred) parameters and give the host class access to the template. Thus, do this:
            
            ```cpp
            // Libarary code
            template <template <class Created> class CreationPolicy>
            class WidgetManager : public CreationPolicy<Widget> {}
        
            // App code
            typedef WidgetManager<OpNewCreator> MyWidgetManager;
            ```
                
            
            (The Widget in the library class’s inheritance allows it to specify the type it works with—it is the Created class.)
            
        *   Combining policy classes in a single host class:
            
            ```cpp
            // Library code
            template <class T, template<class> class CheckingPolicy, template<class> class ThreadingModel>
            class SmartPtr;
        
            // App code
            typedef SmartPtr<Widget, NoChecking, SingleThreaded> WidgetPtr;
            ```
                
            
            *   The most important thing to consider in decomposing a class into policies is that the policies should ideally be orthogonal (they shouldn’t interact).
