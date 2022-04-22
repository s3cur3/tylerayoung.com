---
title: Notes on Game Programming Patterns by Robert Nystrom
categories: ['Game Development', 'Programming']
authors: ['tyler']
layout: post
---

_[Game Programming Patterns](http://gameprogrammingpatterns.com)_ is a game developer’s guide to the design patterns most often useful in that domain. You can [read it online](http://gameprogrammingpatterns.com/contents.html) for free.

- [Chapter 1: Intro](#chapter-1-intro)
- [Design Patterns Revisited](#design-patterns-revisited)
  - [Chapter 2: The Command Pattern](#chapter-2-the-command-pattern)
  - [Chapter 3: The Flyweight Pattern](#chapter-3-the-flyweight-pattern)
  - [Chapter 4: The Observer Pattern](#chapter-4-the-observer-pattern)
  - [Chapter 5: The Prototype Pattern](#chapter-5-the-prototype-pattern)
  - [Chapter 6: (Problems with the) Singleton Pattern](#chapter-6-problems-with-the-singleton-pattern)
  - [Chapter 7: The State Pattern](#chapter-7-the-state-pattern)
- [Sequencing Patterns](#sequencing-patterns)
  - [Chapter 8: The Double Buffer Pattern](#chapter-8-the-double-buffer-pattern)
  - [Chapter 9: The Game Loop Pattern](#chapter-9-the-game-loop-pattern)
  - [Chapter 10: The Update Method Pattern](#chapter-10-the-update-method-pattern)
- [Behavioral Patterns](#behavioral-patterns)
  - [Chapter 11: The Bytecode Pattern](#chapter-11-the-bytecode-pattern)
  - [Chapter 12: The Subclass Sandbox Pattern](#chapter-12-the-subclass-sandbox-pattern)
  - [Chapter 13: The Type Object Pattern](#chapter-13-the-type-object-pattern)
- [Decoupling Patterns](#decoupling-patterns)
  - [Chapter 14: The Component Pattern](#chapter-14-the-component-pattern)
  - [Chapter 15: The Event Queue Pattern](#chapter-15-the-event-queue-pattern)
  - [Chapter 16: The Service Locator Pattern](#chapter-16-the-service-locator-pattern)
- [Optimization Patterns](#optimization-patterns)
  - [Chapter 17: Data Locality (I.e., data-oriented design)](#chapter-17-data-locality-ie-data-oriented-design)
  - [Chapter 18: The Dirty Flag (a.k.a. Dirty Bit) Pattern](#chapter-18-the-dirty-flag-aka-dirty-bit-pattern)
  - [Chapter 19: The Object Pool Pattern](#chapter-19-the-object-pool-pattern)
  - [Chapter 20: The Spatial Partition Pattern](#chapter-20-the-spatial-partition-pattern)

Chapter 1: Intro
----------------

*   Architecture is a tradeoff between 3 types of “speed”:
    *   speed of future modifications (making the program flexible),
    *   execution speed (don’t want to overdo it and get into Java land, where there are 15 layers to get to a piece of code that actually does something)
    *   short term development speed
*   Ideally, you’re decoupling and introducing abstractions in the places most likely to change, or most likely to need the future flexibility, but beware of overdoing it ([YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it))

Design Patterns Revisited
-------------------------

New takes on a few “classic” patterns described in the original book on the subject, [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns) by the “Gang of Four”: Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides.

### Chapter 2: The Command Pattern

> Encapsulate a request as an object, thereby letting user paramaterize clients with different requests, queue or log requests, and support undoable operations.

*   “A command is a reified method call”—in other words, a callback/first-class function/function pointer/etc.
*   E.g., instead of binding button presses to actions directly, you could swap out the direct method calls for usages of a command object; then, the user could configure what happens when each button is pressed
    *   Adds a layer of indirection between the press and the method call
    *   Similarly, you could make the object being acted (e.g., a character) a parameter of the command; then, that same command class could be used to make any character do an action (could be used by the AI or a real player)
    *   This is a bit like currying in Javascript—you wait to apply some parameters until just the right moment
*   Can also use this to decouple input from execution—input handler or AI generates commands, places them in a queue, and other code picks the commands off and executes them at its leisure
*   This is also how you implement undo/redo: you’ve got a long list of commands you’ve received, and each one has an `execute()` and an `undo()` method (since ideally it can apply _just_ the changes it needs, rather than remembering the full state of the object at each time); then, you move forward in the queue by calling `execute()`, and backward in the queue by calling `undo()`
    *   This same technique is useful for doing replays: you record the commands performed each frame, and to replay, you just run the normal game, executing the pre-recorded commands at the right time

### Chapter 3: The Flyweight Pattern

> Use sharing to support large numbers of fine-grained objects efficiently.

*   Suppose you have a million quite heavyweight objects, all of which are similar in some regards (e.g., a forest of trees which all share the same mesh & textures)
    *   Key observation: even though there are thousands of objects, they’re mostly similar
    *   Pull out the data they have in common and move it into a separate class (e.g., TreeModel)
    *   Only load _one_ of those (and make it const, for Pete’s sake!)
    *   All individual trees get a shared reference to the shared model
    *   Shared stuff needs to be context free (hence const-ness)
*   Common application: instead of a bunch of functions to get particular properties, each of which has a giant switch on an enum (which has the problem that info about any given enum is spread way, way out), replace the enum entirely with a flyweight: one object of each type, which has its properties stored directly
    *   Then, you store a pointer to the flyweight, rather than the enum
    *   Advantage: hella less coupling; get to work with real objects that simply tell you about themselves
    *   Advantages of OOP without creating a bunch of objects

### Chapter 4: The Observer Pattern

> Define a one-to-many dependency between objects so that when one object changes state, all its dependencies are notified and updated automatically.

*   Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified & updated automatically
    *   Example: physics code notifying the achievement system—you really don’t want to couple the two, so the physics code says “I don’t know if anyone cares, but this thing happened; do with that what you will”
*   Typical parameters to the notification call are the object that sent the notification and a generic “data” field
*   Cost of this is negligible—but note that this occurs synchronously, so you have to be careful about blocking the main thread (if you need asynchronous, use [the Event Queue pattern](#chapter15-theeventqueuepattern) in Chapter 15)
*   If you don’t want to keep a list of observers separate from the objects themselves, use an intrusive linked list, threading the subject’s list through the observers themselves
*   Note that you’ve got a design problem if two observers observing the same subject have some kind of ordering dependency between them
*   You may want a “last breath” notification if you’re observing an object that could be destroyed
*   Advantages: Extremely low coupling
*   Disadvantages: all coupling is visible only at runtime—hard to reason statically about who’s receiving notifications
    *   In general, you don’t want to use this within a subsystem (where you typically need to reason about _both_ sides of the communication to understand how something works), where you actually care; use it to communicate between independent subsystems
*   Could use a lighter weight solution: register member function pointers as observers instead of implementing an interface

### Chapter 5: The Prototype Pattern

> Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

*   Key idea: Objects can spawn other objects similar to themselves.
*   Give base class (e.g., Monster) an abstract `clone()` method (factory method for duplicating “this” object)
*   This makes it easy to spawn new instances of each subtype ad nauseum
    *   Spawner objects internally hold an instance of a subtype—a hidden one used as a template for all spawning
    *   Doesn’t have to just spawn different _types_… can also spawn instances of the same type (e.g., Ghost) with different characteristics (fast Ghosts, slow Ghosts, etc.)—totally flexible!
    *   Similarly, could do: `Spawner * ghostSpawner = new SpawnerFor<Ghost>();`
*   Most useful in the real world for data modeling
    *   Just like we want to reuse code rather than copying & pasting, artists want to do the same
    *   Scenario: designers need to specify attributes for monsters and items.
        *   Commonly use JSON
        *   Add support for a “prototype” field to support data reuse—any properties not in the object being declared fall back to its prototype
        *   E.g.,
            
                {  
                    "name": "goblin grunt",  
                    "minHealth": 20,  
                    "maxHealth": 30,  
                    "resistances": ["cold", "poison"],  
                    "weaknesses": ["fire", "light"]  
                }
                {  
                    "name": "goblin wizard",  
                    "prototype": "goblin grunt",  
                    "spells": ["fireball", "lightning bolt"]  
                }
                
            
        *   No need to set up any kind of “abstract” prototype… just delegate to the first concrete type we declared
    *   This is a particularly good fit in games with one-off, special entities throughout the world
    *   Makes it extremely cheap to create many tiny variations that make a world seem believable (e.g., don’t just give the player a basic sword 100 times throughout the game… give them the sword of extra damage, the sword of extra attack, the sword of lightning bolts, etc.)

### Chapter 6: (Problems with the) Singleton Pattern

> Ensure a class has one instance, and provide a global point of access to it.

*   Singleton is an easy way out to the question of “how do I get an instance?”
*   Some (rare) classes cannot operate correctly if there’s more than one instance (typically classes that interface with an external system that has its _own_ global state—e.g., file writers, which need to protect against conflicting manipulations of the same file running asynchronously)
*   Thread-safe initialization in C++11:
    
        static FileSystem * FileSystem::instance()  
        {
            static FileSystem * s_instance = new FileSystem();  
            return s_instance;  
        }
        
    
*   Features:
    *   Initialized at runtime (saves you from static initialization hell… but beware the issue of not knowing exactly _when_ it might be initialized!)
    *   Instance isn’t created if no one uses it
    *   You can subclass it—e.g., different FileSystem initialization per platform, where the `instance()` method is defined `#if PLATFORM == PLAYSTATION`, `#if PLATFORM == NINTENDO`, etc.
*   Long-term issues:
    *   It’s a global variable
        *   Harder to reason about the code used by individual functions (ruins function purity)
        *   Encourages coupling—the instance of our AudioPlayer is visible globally, so why _not_ `#include` it in the middle of our physics code?
            *   Controlling access to instances _also_ controls coupling!
        *   Not concurrent—prone to performance issues at best, synchronization bugs, deadlocks, race conditions, etc. at worst
    *   “Solves” two problems—ensuring a single instance, _and_ allowing global access—even if you only needed _one_ of those
        *   E.g., logging class—only needed global access. What happens when you want each subsystem to write its own log? Have to go change every call to `Log::instance()`!
*   Singletons are a way to avoid answering the question of how you architect your code
*   Lazy initialization takes away control
    *   If you have a perf issue, need to be careful about when it’s initialized
    *   Can’t control where in the heap memory gets allocated
    *   Can solve this with static initialization… but if you wanted that, why not just make a static class in the first place and do away with the `instance()` calls altogether?!
*   What to do instead
    *   See if you need the class at all. Many singletons are “managers” whose functionality should be baked into the thing they manage.
    *   To ensure single instance: declare a `static bool s_is_instantiated`; `assert()` in the constructor that it’s false
    *   To provide convenient access:
        *   Pass it in! (Dependency injection)
        *   Get it from a base class (e.g,. have `GameObject` create a static instance so you don’t have to pass it to individual subclasses)
            *   This is the “Subclass Sandbox”
        *   Get it from something already global
        *   Get it from a Service Locator

### Chapter 7: The State Pattern

> Allow an object to alter its behavior when its internal state chnages. The object will appear to change its class.

*   Rather than an increasingly complicated series of nested conditionals, use a finite state machine.
    *   E.g., the allowed actions for a character object change depending on its current state: standing, jumping, ducking, diving all allow different action
    *   Complex branching and mutable state (fields that change over time) make for very error-prone code
    *   Finite state machine characteristics:
        *   fixed set of states
        *   always in exactly one state at a time
        *   events or input get sent to the state machine
        *   each state has a set of transition, and inputs can cause it to move to a different state
    *   Naive implementation: a state enum, and a switch/case block within each of your methods (e.g., `handle_input()`, `update()`, etc.)
        *   Downside: all the logic for a particular state gets spread out across many methods
    *   Improvement: use dynamic dispatch (i.e., a virtual method)!
        *   use a base `CharacterState` class, and create a separate class for each state (`WalkingState`, `JumpingState`, etc.) which implement the methods you need (`handle_input()`, `update()`, etc.)
        *   Then, the `Character` object just has a state _member_ variable which it delegates to—so `Character::handle_input()` calls `m_state->handle_input()`, etc.
        *   Changing state is just a matter of pointing your state member to a new state object
        *   Features:
            *   Keeps all the logic related to a certain state together
            *   Allows you to have member variables private to a given state
        *   Alternatively, if your state objects have no fields, only a single static method, you can replace this with a state function pointer
        *   How do you get a state object?
            *   Create a single, static object and reuse it (if it has no fields beyond its virtual method implementations)—this is \[the Flyweight pattern\]\[Chapter 3: The Flywheel Pattern\] (Chapter 3)
            *   If you need to instantiate the states, have the states’ virtual `handle_input()` method return a pointer to the new state you should transition to
    *   Further improvement: Enter & exit actions
        *   Get called immediately before/after changing state
        *   Give you a place to consolidate code that has to occur on transitions (regardless of what you’re going to/from)
        *   May want to give them your main object (e.g., your `Character`) as a parameter so they can manipulate it
*   Issue with FSMs: they’re a poor fit for more complex problems (like game AI)
    *   Possible solutions:
        *   Concurrent state machines (e.g., one state for the character itself, one for its equipment, etc.)
            *   If you have _n_ states for the character, and _m_ states for its equipment, it requires _n_ × _m_ states, but with concurrent state machines, it’s only _n_ + _m_
            *   Probably want a way for one state machine to consume an input so that the other doesn’t receive it (preventing both from erroneously responding to the same input)
        *   Hierarchical state machines
            *   Use inheritance to share implementation between similar states (e.g., standing, walking, running, sliding could all inherit from an “on the ground” state)
            *   Thus, you get superstates and substates
            *   If a substate doesn’t handle a given input, it’s responsible for passing it up the chain
            *   Beware: inheritance is powerful, but it leads to very strong coupling. “It’s a big hammer, so swing it carefully.”
            *   If you don’t want to use inheritance, can model a chain of superstates explicitly with a _stack_ of states—go through the stack, and keep offering the input until one of the states in the stack consumes it
*   More issues: lack of history
    *   An FSM knows which state it’s in, but no where it came from—no good way to go back to a previous state
    *   Where a single FSM has a single pointer to a state, a pushdown automaton has a stack of them
    *   Where an FSM _always_ replaces a new state during a transition, a pushdown automaton can do that, or it can instead push a new state onto the stack, or pop the topmost off (transitioning to the previous state)
    *   E.g., you transition from standing to firing; when firing is done, you need to know where you came from (a pop operation)

Sequencing Patterns
-------------------

Tools for dealing with the game clock, and the nature of games being broken into cycles/frames.

### Chapter 8: The Double Buffer Pattern

> Cause a series of sequential operations to appear instantaneous or simultaneous.

*   Classic use case is computer graphics, of course—can’t have the video driver reading from the buffer at the same time you’re writing to it, or you get tearing
    *   State gets modified incrementally throughout the game loop
    *   May be “read” by the video driver in the middle of modification
    *   Need to prevent the video driver from reading our half-written copy
    *   Note that the swap operation must be atomic (or else you’re right back where you started!)
*   Typical implementation: two separate buffers in memory
    *   The “swap” is just changing a pointer to point to the new “active” buffer
    *   Important to note that outside code can’t store persistent pointers to the buffer
    *   Can be useful to note that existing data in the “write” buffer comes from two frames ago—can use this to do motion blur (huh!)
*   Not just for graphics! Consider an AI, where you want all actors to appear to update simultaneously
    *   In this case, each actor has an `update()` method, and a `swap()` method
    *   In the `swap()` method, each actor changes their “next” state to their “current,” and resets their “next” state to default
    *   Then, the “manager” object calls `update()` on all actors, then `swap()` at the end of the frame
    *   This is a case where we _copy_ the data between buffers (for _every_ object!) rather than simply swapping a pointer
    *   Note that data in the active buffer is only a frame old (this could be useful)

### Chapter 9: The Game Loop Pattern

> Decouple the progression of game time from user input and processor speed.

*   At it’s most basic, this looks like:
    
        while(true)  
        {
            processInput();  
            update(); // Cf. chapter 10 on the Update pattern  
            render();  
        }
        
    
*   Each frame, your game needs to process user input (if applicable), but of course you don’t want to _wait_ for input
*   Old games were designed for specific hardware, so the game did just enough work to run at the desired speed.
    *   Aside: Older PCs came with “turbo” buttons: they were too fast to play old games, so you would turn the turbo mode _off_ (it was _on_ by default) to slow down the PC and make old games playable!
*   Game loop’s job is to run the game at a consistent speed regardless of the underlying hardware.
    *   Tracks the passage of time each frame in order to control the rate of play
*   **Bad implementation**: variable/fluid time step
    *   Make the time steps grow or shrink to match the real processing time required
    *   Each cycle takes a certain amount of real time to process; just make the game time advanced be less than or equal to this!
    *   An object moving across the screen has its velocity scaled by the elapsed time each frame
    *   Good: the game is playable on hardware of differing speeds, and faster machines get smoother gameplay (better than simulating at a low frame rate and sleeping until you need to render the next frame)
    *   Bad: the game is non-deterministic and unstable
        *   An object moving at the same speed from the same starting location—but at _different_ frame rates—will end up in a different place due to accumulated floating point rounding errors!
        *   This primarily affects physics and networking
*   **Better implementation**: Fixed update time, variable rendering
    *   _Update_ using a fixed time step, but _render_ only when we have time
    *   Each cycle, we “catch up” to the real time elapsed using a series of fixed steps
        
            double previous = getCurrentTime();  
            double lag = 0; // how far game clock is behind real world  
            while(true)  
            {
                double current = getCurrentTime();  
                double elapsed = current - previous;  
                previous = current;  
                lag += elapsed;  
                processInput();
            
                // Update in fixed-length intervals until we (nearly) catch up to real time  
                while(lag >= MS_PER_UPDATE)  
                {
                    update();  
                    lag -= MS_PER_UPDATE;  
                }
                render();  
            }
            
        
        *   Aside: [Seriously, game time should be a double, not a float](https://randomascii.wordpress.com/2012/02/13/dont-store-that-in-a-float/)
    *   Important points:
        *   Time step is no longer locked to frame rate; typically want to update faster than 60 FPS, but not _so_ short that slow machines can’t get through an `update()` cycle fast enough
            *   Safeguard against a too-slow `update()` by making the game bail after too many iterations through the loop
        *   Rendering is typically much cheaper than updating, so by taking rendering out, you buy yourself a lot of CPU time
    *   The challenge: rendering often comes _between_ `update()` ticks (i.e., there is residual lag when `render()` is called)
        *   Left uncontrolled, this makes for stutter
        *   Solution: pass `lag / MS_PER_UPDATE` into `render()` and have it extrapolate locations based on current velocities (might draw a bullet half a frame ahead)
*   Other considerations:
    *   Power usage: may clamp the frame rate to save on battery
*   Further reading:
    *   Glenn Fiedler’s “[Fix Your Timestep!](https://gafferongames.com/post/fix_your_timestep/)”
    *   Koen Witters’ “[deWiTTERS Game Loop](https://www.koonsolo.com/news/dewitters-gameloop/)
    *   [The Unity game loop diagram](https://docs.unity3d.com/Manual/ExecutionOrder.html)

### Chapter 10: The Update Method Pattern

> Simulate a collection of independent objects by telling each ot process one frame of behavior at a time.

*   Processes one frame of behavior at a time for a collection of independent objects
*   Rather than filling your game loop with the logic to update each individual entity type, you have the entities encapsulate their own behavior/logic/animation/etc.
    *   Makes it easy to add/remove entities without modifying external code
    *   Game loop just knows it has a collection of updatable objects (not their concrete types)
    *   Could even populate the entity list from a data file from a level designer, rather than having them hard-coded
    *   [The State pattern](#chapter7-thestatepattern) (Chapter 7) can help here
*   Potential issues
    *   Complicates the logic of how objects behave by introducing state
        *   Alternative: if you don’t want to have to break up your object’s behavior by thread, you could put objects on separate (lightweight) threads executing at the same time; objects could pause/yield rather than returning, leading to more imperative code with less boilerplate for state and per-frame logic
    *   This requires each object’s behavior is (mostly) independent of the others—no guarantees about relative ordering of updates
    *   If order matters—you want to guarantee Object B is looking at the _previous_ frame’s state for Object A when it updates—you need a Double Buffer; this is usually not an issue, though
    *   You have to be careful when adding/removing objects _during_ `update()` iterations—may want to mark as “dead” and defer deletion until you’re done walking the list
    *   If your Game Loop uses a variable time step, you probably want to pass the elapsed time since the last frame into `update()`
*   Decisions to make
    *   Where does the update method live? Options include:
        *   an Entity class (requires all subclasses to inherit from it, which can be brittle and limiting)
        *   A component class (see the Component pattern below); this decouples parts of a single entity from each other (rendering, physics, AI, etc. can all be separate components)
        *   A delegate class—see [the State pattern](#chapter7-thestatepattern)’s delegation (Chapter 7) and [the Type Object pattern](#chapter13-thetypeobjectpattern) (Chapter 13)
    *   How to handle inactive objects—may just want a separate list containing only active ones that need updating. Iterating objects that you then skip leads to cache misses.

Behavioral Patterns
-------------------

Strategies for specifying a large amount of behavior (e.g., for game entities) in a maintainable way. Bytecode and type objects focus on turning behavior into _data_, rather than code, while a subclass sandbox focuses on abstracting out a “tool box” of methods such that subclasses can implement their behavior with minimal coupling to the outside world.

### Chapter 11: The Bytecode Pattern

> Give behavior the flexibility of data by encoding it as instructions for a virtual machine.

*   Motivation: want to ship behavior as data, not code, so that it’s:
    *   Easy to create for nontechnical users
    *   Easy to patch later
    *   Fast to iterate on (no huge recompile necessary)
    *   Moddable by the community
*   The Gang of Four’s [Interpreter](https://en.wikipedia.org/wiki/Interpreter_pattern) pattern won’t do, because it’s slow: you create a huge number of small objects (lots of memory used), then chase their vtables all over the place (blowing your instruction cache)
*   In contrast, bytecode is super dense, linear, and low level… but still very safe like the Interpreter pattern
    *   Other advantages:
        *   You control the size of the stack (and thus the memory usage)
        *   You can control how long the VM is given to execute each frame
    *   Downsides:
        *   Requires you to either have users edit text files (which you later compile down to bytecode) or create a UI in which to visually edit it
        *   Language has a way of growing out of control
        *   Debugging is hard (and on _you_!)
*   Implementation: use a stack machine to push instructions and parameters; as you execute instructions, they pop parameters off the stack.
    *   E.g., add the average of wizard 0’s agility and wisdom to its health
        *   In code, this would be:
            
                setHealth(0, getHealth(0) +  
                    (getAgility(0) + getWisdom(0)) / 2);  
                
            
        *   In bytecode, this translates as:  
            
              
            
            Instruction
            
            Stack state
            
            Comment
            
            LITERAL 0
            
            \[0\]
            
            Wizard index
            
            LITERAL 0
            
            \[0, 0\]
            
            Wizard index
            
            GET\_HEALTH
            
            \[0, 45\]
            
            getHealth()
            
            LITERAL 0
            
            \[0, 45, 0\]
            
            Wizard index
            
            GET\_AGILITY
            
            \[0, 45, 7\]
            
            getAgility()
            
            LITERAL 0
            
            \[0, 45, 7, 0\]
            
            Wizard index
            
            GET\_WISDOM
            
            \[0, 45, 7, 11\]
            
            getWisdom()
            
            ADD
            
            \[0, 45, 18\]
            
            Add agility and wisdom
            
            LITERAL 2
            
            \[0, 45, 18, 2\]
            
            Divisor
            
            DIVIDE
            
            \[0, 45, 9\]
            
            Average agility and wisdom
            
            ADD
            
            \[0, 54\]
            
            Add average to current health
            
            SET\_HEALTH
            
            \[ \]
            
            Set health to result
            
*   Note that if you need to support multiple value types (i.e., you can’t get by with ints alone), you probably want to represent values internally as a tagged variant

### Chapter 12: The Subclass Sandbox Pattern

> Define behavior in a subclass using a set of operations provided by its base class.

*   Scenario: Base class has lots of derived classes with lots of overlapping behavior, and you don’t want those classes coupled to the rest of the engine
*   Idea: define a superclass that encapsulates most/all contact with the rest of the engine, and allow subclasses to implement their behavior using the (protected) “primitive” methods the superclass provides
    *   Subclasses implement a single, virtual, abstract, protected , sandbox method (e.g., `activate()`) whose behavior is defined in terms of the superclass’s provided functions
    *   E.g., superclass provides `playSound()`, `emitParticles()`, `movePlayer()`, etc.
    *   This allows you to limit coupling between the engine and your (potentially huge) set of classes to just _one_ class (rather than having _all_ subclasses coupled to the engine)
    *   This is a variation of the [Facade](https://en.wikipedia.org/wiki/Facade_pattern) pattern: we’re hiding a number of engine components behind a single, simplified API
    *   This is a role reversal from the [Template Method](https://en.wikipedia.org/wiki/Template_method_pattern) pattern, which has the _base_ class implement a single operation via primitives provided by _child_ classes
*   Downsides:
    *   Shallow-but-wide class hierarchy makes it hard to change the base class without breaking something (brittle base class problem)
    *   Base class can grow into a huge API
*   Decisions to make:
    *   Which operations to provide
        *   You get the most leverage (in terms of adding complexity to the base class vs. just definining the behavior in the subclasses) by providing operations used in lots of subclasses
        *   Calls that modify external state are best in the base class due to increased visibility
        *   May want to wrap even simple forwarding calls in the base class if there is state that you don’t want exposed to subclasses
    *   Provide methods directly, or via sub-objects?
        *   Might want, e.g., a `getSoundPlayer()` on the base class, so that you can encapsulate the sound-related API into a smaller, more easily maintainable unit
    *   How to initialize the base class with its required state
        *   **Bad**: Passing it to the base class ctor sucks: it’s a maintenance headache that requires you to modify every ctor of child classes if you ever add/remove parameters
        *   **Bad**: Two-stage initialization—a normal ctor for the subclass, plus a required, non-virtual `init()` call in the subclass—sucks because it’s easy to mess up
        *   **Better**: Two-stage initialization _via_ a factory method like:
            
                Superpower * createSkyLaunch(ParticleSystem * ps) {  
                    Superpower * power = new SkyLaunch();  
                    power->init(ps);  
                    return power;  
                }
                
            
        *   **Best?**: Make the state static! Initialize the base class once, before subclasses are instantiated
        *   **Best?**: Use a [Service Locator](#chapter16-theservicelocatorpattern) (see Chapter 16)—the base class can pull in the state it needs. This is the only one that _doesn’t_ place the burden of initialization on the surrounding code.

### Chapter 13: The Type Object Pattern

> Allow the flexible creation of new “classes” by creating a class, each instance of which represents a different type of object.

*   Motivation: we want lots of different monster “breeds,” each of which has its own set of characteristics (starting health, attack types, etc.)
    *   OOP way: A huge class hierarchy, with the characteristics defined in code. This is way too rigid—requires a _programmer_ if you want to change the simplest attributes.
    *   Alternative: Each instance of `Monster` _has-a_ `Breed`; that `Breed` defines the attributes
        *   This lifts a huge portion of the type system out of code and into (runtime-defined) data
        *   Allows us the flexibility of supporting, e.g., downloadable content with new types!
        *   We’re typically more limited by the speed of content authoring than the runtime speed of these sorts of objects, so this is a win
        *   [The Prototype pattern](#chapter5-theprototypepattern) (Chapter 5) addresses the same problem (sharing data & behavior between objects) in a different way.
        *   This is related to [the Flyweight pattern](#chapter3-theflyweightpattern) (Chapter 3), but instead of trying to save memory, this is focused on organization and flexibility
*   Instantiate one instance of `Breed` for each conceptual breed, and give `Monster`s a reference to the `Breed` that defines their type
    *   In this case, `Breed` is the “type object” `Monster` is said to be the “typed object”
    *   This is analogous to having a vtable where we look up class attributes! (We’re essentially implementing our own virtual methods)
*   Downsides:
    *   Defining behavior is harder than in code. Options include:
        *   Having a limited selection of pre-defined behaviors, and allowing type objects to choose one
        *   Use the Interpreter or [Bytecode](#chapter11-thebytecodepattern) (Chapter 11) patterns to define behavior in the data
    *   Type objects require manual management—have to make sure they exist before the typed objects you create
*   Implementation details:
    *   Probably do _not_ want clients of `Monster` to have to deal with managing the corresponding `Breed` too. Instead, give `Breed` classes a factory method to construct a `Monster` of that type
        *   Bonus: the type object can control the allocation here, e.g., to use an [Object Pool](#chapter19-theobjectpoolpattern) (Chapter 19)
    *   Supporting inheritance for sharing data
        *   E.g., breeds can specify a parent type; if breed definition gives a health of 0, look it up in the parent
        *   Rather than having to walk the inheritance chain at runtime, probably want to do “copy-down delegation” (copy the relevant attributes once at construction time)
        *   E.g.,
            
                {  
                  "Troll": {  
                    "health": 25,  
                    "attack": "The troll hits you!"  
                  },  
                  "Troll Archer": {  
                    "parent": "Troll",  
                    "health": 0, // Inherit from parent  
                    "attack": "The troll archer fires an arrow!"  
                  },  
                  "Troll Wizard": {  
                    "parent": "Troll",  
                    "health": 0, // Inherit from parent  
                    "attack": "The troll wizard casts a spell on you!"  
                  }
                }  
                
            
*   Design decisions:
    *   Encapsulate or expose the type object?
        *   Pro: Encapsulation allows the typed object to selectively override the behavior of its type (e.g., as a monster gets low health, it might override its attack with “flail weakly”)
        *   Con: You have to forward all calls down to the type object as necessary
    *   Can the type change?
        *   Pro: Super flexible. E.g., instead of creating a new object when a dead enemy turns into a zombie, you just change its underlying type object)
        *   Con: hard to reason about whether your assumptions will be met (e.g., will current health ever be above your `Breed`’s starting health?)
    *   Inheritance: Odds are good you need to support _some_ inheritance; stay away from multiple inheritance, though—your designers will never wrap their heads around it (even if, by some miracle, you do)

Decoupling Patterns
-------------------

One of the hardest parts of programming is making your code easy to change in the future. This section focusing on decoupling parts of your codebase from each other—both in terms of static linkage and chronological ordering of events.

### Chapter 14: The Component Pattern

> Allow a single entity to span multiple domains without coupling the domains to each other.

*   Motivation: Your main player character’s `update()` method can get unwieldy fast. As you process input, update physics, render, etc., you find that not only is your character coupled to your entire game engine, but that you’ve tied what should be separate aspecs of the _character_ to each other.
    *   This is a recipe for no developer on your team fully understanding any aspect of the behavior.
    *   What’s worse, if you want to be more parallelizable, one common way is to split your code across threads based on domain boundaries—AI, sound, rendering, etc. might each have their own cores. Coupling the domains at that point is going to lead to concurrency bugs!
*   Solution: Split the behavior out into `Component` subclasses, such that your `Character` _has-a_ `PhysicsComponent`, `GraphicsComponent`, etc. Then, you reduce `Character::update()` to simply calling each of its _component’s_ `update()`s.
    *   This is related to the Gang of Four’s [Strategy](https://en.wikipedia.org/wiki/Strategy_pattern) pattern, but where a Strategy is generally a stateless encapsulation of an algorithm (defining _behavior_, not _identity_), the Component pattern typically holds state and is part of an object’s identity.
    *   At the extreme end, your “characters” just become a bag of components, with no identity remaining in the character class itself at all. You can create all “characters” via a Factory Method that just stuffs components into a generic `GameObject`.
        *   Even more extreme: _Entity component systems_ do away with the notion of a `GameObject` with a bag of components; instead, game entities are just an ID, and you maintain a separate collection of components which all know the ID they’re attached to—this allows you to add components to an entity without the entity even “knowing.” See [the Data Locality pattern](#chapter17-datalocalityi.e.data-orienteddesign) (Chapter 17) for details.
    *   Bonus: This makes components reusable across classes—you can share AI, physics, etc. between entity types.
    *   Bonus: You can swap a component (e.g., swap the player input component with an AI “input”) and the rest of the components won’t know the difference.
    *   Bonus: Components make it easier to use [the Data Locality pattern](#chapter17-datalocalityi.e.data-orienteddesign) (Chapter 17) to organize your data in the order the CPU will use it
*   Downsides:
    *   Communication between components becomes more challenging.
    *   Managing the way the components occupy memory is more complex
*   How will components communicate? Often, the answer is some amount of _all_ of the following:
    *   By modifying the container object’s state
        *   Good for basic stuff that all objects have, like position & size
        *   Too much shared, mutable state makes code very hard to get right
        *   Makes communication implicit and dependent on the order in which components are processed
        *   Wastes memory if some data isn’t required by _any_ of the object’s components (e.g., rendering data in an invisible object)
    *   By calling each other directly
        *   Good for closely related domains (e.g., animation & rendering)
        *   Tightly couples the two components
    *   By passing messages
        *   Good for “less important” communication (e.g., send a message to play a sound after a physics event)
        *   Give the `Component` class an abstract virtual `receive()` method, and your container a `send()` method; the container is responsible for broadcasting all messages to all its components.
            *   This is an instance of the Gang of Four’s [Mediator](https://en.wikipedia.org/wiki/Mediator_pattern) pattern.

### Chapter 15: The Event Queue Pattern

> Decouple when a message or event is sent from when it is processed.

*   Motivation: Consider the sound system in a game. `playSound()` might have to load data from disk, making a synchronous (blocking) implementation hugely unfriendly to threaded code (not to mention problems caused by client code requesting the same sound to be played in the same frame).
*   Solution: Instead, `playSound()` can simply enqueue the request for processing during the sound system’s `update()` (on its own thread).
    *   By decoupling _receiving_ a request from _processing_ it, we can take care of all these problems.
*   This is related to the [Command](#chapter2-thecommandpattern) (Chapter 2) and [Observer](#chapter4-theobserverpattern) (Chapter 4) patterns, in that it decouples the recipient of a message from its sender. But, the event queue uniquely decouples them _in time_.
    *   Advantage of the event queue: the code that “pulls” from the queue has more control over when the queue will be handled.
    *   Disadvantage: the sender doesn’t get a response if/when its request is handled.
*   Potential pitfalls
    *   Because the state of the world might change between when a request is sent and when it’s processed, your requests often have to include more data than in a synchronous system.
    *   You have to be careful to avoid feedback loops—a good rule of thumb is to avoid sending events from within code that handles them.
*   Implementation: use a [ring buffer](https://en.wikipedia.org/wiki/Circular_buffer) for a fixed-size queue
*   Events vs. messages
    *   Events describe something that _already_ happened that other code might want to respond to (like an asynchronous [Observer](#chapter4-theobserverpattern) from Chapter 4)
        *   Often have multiple listeners
        *   Events are often broadcast
        *   These queues tend to be more globally visible
    *   Messages act as a request to do something in the future (like an asynchronous API for a service)
        *   Typically have a single listener
        *   If you _don’t_ have a single listener, you’re probably enqueuing messages without caring who processes it (only _how_ it’s processed), in which case you have something more like an async service location
*   Queue types
    *   Single listener: fully encapsulated, don’t have to worry about contention
    *   Broadcast queue (like an event system): events can get dropped (no applicable listener), may need to filter events (allow listener to subscribe to only a subset of events), have to deal with thread contention
    *   Work queue: multiple listeners like a broadcast queue, but only each item gets doled out to exactly one listener; have to figure out how to schedule jobs
*   Related: if you have a collection of [state machines](#chapter7-thestatepattern) (Chapter 7) receiving messages via async queues, you’re using the _actor model of computation_. Each state machine’s queue that it reads from is called its mailbox.

### Chapter 16: The Service Locator Pattern

> Provide a global point of access to a service without coupling users to the concrete class that implements it.

*   Decouples client code from both _who_ and _where_ the service is (concrete implementation and how to get the instance of it)
*   When to use it
    *   Can be overkill if you can simply pass the required dependency in as a parameter
    *   For unrelated systems, it makes sense to _not_ pass them as parameters (e.g., logging system used by the rendering engine)
    *   Services that are fundamentally singular in nature (like audio) are a good fit—Service Locator acts as a more flexible, more configurable cousin of [Singleton](#chapter6-problemswiththesingletonpattern) (Chapter 6)
*   Requirements for implementation
    *   The service can’t control who’s using it or when in the frame—if your class expects to only be used in a certain context, don’t expose it globally like this
*   Advantage over a global: Can change the concrete implementation seamlessly
    *   Allows easy decorators
        *   E.g., you have an `Audio` interface implemented by your `ConsoleAudio` class; by default, that’s what your initialization code injects into the service locator, and what the service locator in turn provides client code.
        *   But, you could easily write a wrapper for the `Audio` interface to log relevant method calls like this:
            
                struct LoggedAudio : public Audio {  
                    LoggedAudio(Audio &wrapped) : m_wrapped(wrapped) { }
                
                    virtual void playSound(int sound_id) {  
                        log("playSound()");  
                        m_wrapped.playSound(sound_id);  
                    }
                
                    Audio &m_wrapped;  
                };  
                
            
        *   Then, you simply decorate the existing service and swap out the implementation that your service locator is giving the world: `Locator::provide(LoggedAudio(Locator::getAudio()));`
*   Design decisions:
    *   How is the service located in the first place?
        *   If the service locator itself creates the service, you guarantee it’s always available, but lose flexibility (changing the service requires recompiliing).
        *   If outside code registers it, you control the construction (the service locator itself often doesn’t know enough to construct services), and you can swap out services while the game is running
        *   If you configure which concrete implementation to use at runtime (via a config file), you can swap implementations without compiling, and the same codebase can support multiple implementations simultaneously, but you add a lot of complexity.
    *   What to do if the service can’t be located
        *   Force client code to deal with the possibility of a null pointer (ugh)
        *   Crash (assert the service is always available)
        *   Return a “null” service (a placeholder implementation of the service’s API)—means the game will keep running, but it’s harder to debug an unintenionally missing service
    *   How broadly available is the service?
        *   Could make it available just to a single class and its descendents (via protected methods)

Optimization Patterns
---------------------

### Chapter 17: Data Locality (I.e., data-oriented design)

> Accelerate memory access by arranging data to take advantage of CPU caching.

*   Since 1980, CPU speeds have increased roughly 10,000×, while memory speeds have only increased about 10×
*   If your super-fast CPU is stuck waiting on super-slow memory to read, it stalls out and your performance takes a nosedive
    *   You need to optimize for working with cache lines—organize data structures so that the things you’re processing in a sequence are sequential in memory
        *   Note that this _does_ assume we’re talking about processing data within a single thread. If two threads are modifying the data, you want it in two _different_ cache lines (since they would otherwise need to synchronize—slow!)
    *   In the first example from the chapter, there was a 50× perf increase… just by refactoring how we access the data
    *   In performance critical code, cache misses look like this:
        
            for(int i = 0; i < NUM_THINGS; ++i) {  
                sleepFor500Cycles();  
                things[i].doStuff();  
            }
            
        
    *   Easy perf win: remove the massive wait!
*   Require restructuring your code to _remove_ abstractions: don’t want to have to follow pointers (goodbye inheritance, interfaces, etc.), and especially don’t want to look up vtables for virtual method calls
    *   This is a real cost on maintainability!
    *   Profile first to ensure your perf problems are caused by cache misses
        *   Cachegrind is a free option for doing so
*   Three examples:
    *   First, a note: these examples use [the Component pattern](#chapter14-thecomponentpattern) (Chapter 14). While it’s a common targets for this optimization (because it’s central to the game loop… and having your code in components make it _easier_ to implement!), _all_ performance-critical code needs to be aware of data locality.
    *   Prefer to traverse _contiguous_ arrays
        *   Compare this:
            
                for(int i = 0; i < numEntities; ++i) {  
                    entities[i]->ai()->update();  
                }
                for(int i = 0; i < numEntities; ++i) {  
                    entities[i]->physics()->update();  
                }
                for(int i = 0; i < numEntities; ++i) {  
                    entities[i]->renderer()->update();  
                }
                
            
        *   To this:
            
                for(int i = 0; i < numEntities; ++i) {  
                    aiComponents[i].update();  
                }
                for(int i = 0; i < numEntities; ++i) {  
                    physicsComponents[i].update();  
                }
                for(int i = 0; i < numEntities; ++i) {  
                    rendererComponents[i].update();  
                }
                
            
        *   This had a 50× speedup for the author!
        *   Removes chasing pointers (which might be located increasingly farther apart as the heap gets more and more fragmented)
            *   Essentially _every_ `update()` call in the original was a cache miss!
        *   Heuristic: the improved game loop uses no indirection—if you’re using the `->` operator, you’re chasing pointers!
        *   Doesn’t require getting rid of `GameEntity`—it can store pointers into the components arrays. _But!_ The perf-critical game loop can sidestep the indirection and operate directly on the data.
    *   Pack your data; don’t traverse inactive objects
        *   Consider a particle system (an instance of an [Object Pool](#chapter19-theobjectpoolpattern) from Chapter 19) that stores lots of `Particle` objects. Naive implementation of the particle system’s update is:
            
                for(int i = 0; i < numParticles; ++i) {  
                    if(m_particles[i].isActive()) {  
                        m_particles[i].update();  
                    }
                }  
                
            
        *   Two problems here:
            *   Loading inactive particles into the cache wastes data
            *   The `if` is going to cause pipeline stalls due to branch mispredictions.
        *   Thus, the contiguous array here doesn’t help much if the objects you’re _actually_ processing aren’t contiguous.
        *   Fix: _sort_ by the active flag rather than checking it—removes thre branch and packs your data
            
                for(int i = 0; i < activeParticles; ++i) {  
                    m_particles[i].update();  
                }
                
            
            *   But, how to keep the array sorted? Any time a particle gets activated, swap it with the first _inactive_ one; any time one gets _deactivated_, swap it with the last active particle (right before the inactive ones)
                *   Our intuition is that moving this data in memory is slow, but when you measure this, you find it’s cheaper to move the memory in order to keep your cache full.
        *   This does tightly couple the particles to the particle system, but that’s okay here—the don’t make much sense _except_ in the context of the particle system. (Plus, the particle system itself is probably the only class that will be activating or deactivating particles.)
    *   Hot/cold splitting
        *   Suppose your AI component has some data that’s used in every `update()`, and other data that’s only used once in the object’s lifetime (say, when the character is killed).
        *   The one-time-use data takes up space in the cache _every_ frame—causing more cache misses every frame simply because the amount of memory we traverse is larger.
        *   Solution: break the data structure into separate pieces; keep the data used per-frame in the object itself, and move the rest into a sub-object; the object can store a pointer to it, or you can just use parallel arrays to keep the two together.
*   Design Decisions
    *   How to handle polymorphism
        *   Don’t! This is obviously safe, easy, and fast, but it’s obviously inflexible. (Code can become a mess quickly with a bunch of giant `switch` statements and the like.)
            *   One possibility: use [the Type Object pattern](#chapter13-thetypeobjectpattern) (Chapter 13) to keep much of the flexibility of polymorphism without actually using subclasses.
        *   Use separate arrays for each type.
            *   Polymorphism allows us to invoke behavior on objects whose types we don’t know.
            *   If you instead maintain separate, homogenous arrays for each type, you can statically dispatch—no polymorphism required.
            *   Problems: your code has to be aware of every type, and keep track of a bunch of collections.
        *   Use a collection of pointers and just deal with the cache issues.
    *   How to define game entities
        *   Use classes with pointers to their components
            *   Pro: Components can be stored and iterated in contiguous arrays
            *   Pro: Easy to get a given entity’s components
            *   Con: If you move the components in memory (e.g., to keep the active ones at the front of the array), you have to update the component object too
        *   Use classes that contain IDs for their components
            *   Pro: Makes moving components in memory easy
            *   Con: Slower to find a given entity’s components (requires a search/hash/etc.)
        *   Make the entity itself nothing but an ID (double down on [the Component pattern](#chapter14-thecomponentpattern) from Chapter 14)
            *   Pro: Don’t have to manage entity lifetime (the “entity” dies when all its components are destroyed)
            *   Con: Looking components for a given entity is slow
                *   This is especially a problem because components for a given entity often need to interact with each other within the hot loop
                *   Could mitigate this by making the ID just an index in a set of parallel arrays… but then you’re really limited in how you can move objects around within your arrays to keep the live data coherent
                    *   E.g., if you have 100 AI components active, 200 renderer components, and 300 physics components, you’re going to be traversing “dead” data in at least some of those arrays
*   More resources:
    *   [_Data-Oriented Design_](http://www.dataorienteddesign.com/dodmain/dodmain.html) by Richard Fabian
    *   “[Pitfalls of Object-Oriented Programming](https://tylerayoung.files.wordpress.com/2017/01/pitfalls-of-object-oriented-programming-tony-albrecht-gcap-09.pdf)” by Tony Albrecht
    *   “[Data-Oriented Design (Or Why You Might Be Shooting Yourself in the Foot with OOP](http://gamesfromwithin.com/data-oriented-design)” by Noel Llopis
    *   The pattern takes advantage of contiguous arrays of homogenous objects, like the Object Pool pattern (Chaper 19) gives you.

### Chapter 18: The Dirty Flag (a.k.a. Dirty Bit) Pattern

> Avoid unnecessary work by deferring ig until the result is needed.

*   Motivation:
    *   A scene graph stores both the _local_ transform (an object’s position relative to… whatever it’s attached to) and _world_ transform. World transforms are calculated (derived) by walking the parent hierarchy up the scene graph and performing matrix multiplication on each anscestor’s local transform.
        *   E.g., a parrot is attached to a pirate, who’s attached to a crow’s nest, which is attached to a ship, which is positioned relative to the scene root.
            *   Parrot’s world transform matrix = Ship local × Crow’s nest local × Pirate local × Parrot local
    *   We don’t want to have to recalculate the world transform for _every_ object in the hierarchy every time _any_ element moves—we might have lots of moves between any given usage.
        *   Need decouple changing the local transforms from updating the world transforms—only calculate world transforms _right_ before we need to render.
    *   Physics engines do the same thing: an object whose `is_moving` flag is false will be exempt from physics calculations; once they’re touched (and a force is applied to them), their moving flag gets set to true.
*   Solution: Use a “dirty” flag to track when the primary data changes; if the flag is set when the derived data is _needed_, the derived data gets reprocessed an the flag gets cleared; otherwise, the derived data’s previously cached value is used.
*   When to use it
    *   When the primary data has to _change_ more often than the derived data is _used_ (If the derived data is needed every time the primary data changes, there’s no advantage here!)
    *   When incremental updates aren’t feasible (if you could implement, say, a running total counter, that’s much simpler)
*   Implementation concerns
    *   There is a cost to deferring too long
        *   Could get uneven frame times if you wind up doing a big batch of work some frames but not others. (This mirrors issues with garbage collector pauses.)
        *   If something goes wrong, you might fail to do the work at all—e.g., if you’re deferring saving data to disk and you lose power, you’re SOL.
            *   Have to choose auto-save frequency as a balance between doing it too frequenty and not losing too much work if a crash occurs.
    *   The flag _must_ change every time the primary data does—probably want to encapsulate changes to it behind an interface.
    *   The derived data has to be kept around in memory (trades memory for time)
*   Sample code:
    
            void GraphNode::render(Transform parent_world, bool dirty)  
            {
                dirty |= m_dirty;  
                if (dirty)  
                {
                    m_world = m_local.combine(parent_world);  
                    m_dirty = false;  
                }
        
                if (m_mesh) renderMesh(m_mesh, m_world);
        
                for(auto &child : m_children)  
                {
                    child.render(m_world, dirty);  
                }
            }  
        
    
    *   The key insight here is the `dirty` parameter. It will be true if any node _above_ this one in the hierarchy was dirty, so children get marked as dirty recursively “just in time”—no need to recurse through them at the time when a parent’s local matrix gets modified.
        *   But, note that if code _other_ than `render()` needed an up-to-date world transform, this trick wouldn’t work.
*   Design decisions
    *   When to clean the dirty flag
        *   Only when needed: avoids doing unnecessary work, but if the calculation is time consuming, risks a noticable pause. (If it’s too slow, you may need intermittent updates earlier.)
        *   At well-defined checkpoints (e.g., a loading screen or other transition): doesn’t impact the UX, but you lose control over when it happens—if the user doesn’t follow the path you expect (e.g., avoids triggering a load for a long time), you might end up deferring much longer than intended.
        *   In the background: you can tune how often the work is performed, but asychrony requires additional complexity, and you wind up duplicating some effort on work that wasn’t needed.

### Chapter 19: The Object Pool Pattern

> Improve performance and memory use by reusing objects from a fixed pool instead of allocating and freeing them individually.

*   Motivation:
    *   The longer a game runs, the more it fragments the heap (i.e., breaks the available space into smaller chunks, rather than one large open block).
    *   E.g.,  
        
         
        
        State
        
        Heap Visualization
        
        Initial (empty 16 KB)
        
        `[* * * * * * * * * * * * * * * *]`
        
        Allocate an object (4 KB)
        
        `[X X X X * * * * * * * * * * * *]`
        
        Allocate another (8 KB)
        
        `[X X X X Y Y Y Y Y Y Y Y * * * *]`
        
        Free the first
        
        `[* * * * Y Y Y Y Y Y Y U * * * *]`
        
        *   In the final state, we can’t allocate another 8 KB object, even though there are 8 KB free!
    *   Plus, heap allocation is just generally slow!
*   Solution: Allocate a large chunk of _reusable_ memory up front, don’t free it for the life of the game, and reuse it as objects become inactive (not destroyed!) and new ones are needed.
    *   Requires you to create an “in-use” query for the objects.
    *   This is sort of similar to [the Flyweight pattern](#chapter3-theflyweightpattern) (Chapter 3), in that both maintain a collection of “reusable” objects. But, whereas Flyweight objects are shared by lots of clients simultaneously, in this case, the reuse occurs over time, and it’s not the same conceptual object (though it is the same chunk of memory).
*   When to use it
    *   When objects are similar in size. (Any size disparity will mean wasted memory and wasted time traversing it.)
        *   If you want to support different sizes of objects, you probably want separate pools, one per block size. Cf. Andrei Alexandrescu’s [CppCon 2015 talk on `std::allocator`](https://www.youtube.com/watch?v=LIb3L4vKZ7U) (talks about composability)
    *   When objects encapsulate resources that are slow to acquire (like a database connection) and could be reused
*   Keep in mind
    *   Have to deal with the case where your fixed number of allocated objects gets all used up. Options include:
        *   Prevent it entirely through careful tuning. (This is tedious, error prone, and may require you to allocate a lot of memory for slots only needed in edge cases.)
        *   Just don’t create the object. (May require all client code to handle this eventuality.)
        *   Forcibly kill an existing object (e.g., the one closest to being done anyway). If the disappearance of an existing object would be less noticeable than the absence of a new one, this might be the best choice.
    *   You don’t get debugging tools for free (like writing over recently freed memory with `0xdeadbeef`). Worse, the memory for newly “created” objects recently held a real instance of the same type, so in its “uninitialized” state, it still looks pretty much like a valid instance of the type! (You probably want to create a debugging mode where you write over the data in instances of the object as you free/allocate them.)
    *   You might be getting in a fight with your garbage collector. (If you’re using this in a garbage collected language, make sure you clear any references a pooled object holds to other objects when it becomes inactive.)
*   Sample code
    
        void ParticlePool::create(double x, double y,  
                          double x_vel, double y_vel,  
                          int lifetime)  
        {
          for (int i = 0; i < POOL_SIZE; i++)  
          {
            if (!m_particles[i].inUse())  
            {
              m_particles[i].init(x, y, x_vel, y_vel, lifetime);  
              return;  
            }
          }  
        }
        
    
    *   Note that the pooled type’s `init()` does the work of a typical constructor.
    *   Objects in the pool must be initialized to a “not in use” state (e.g., by the pooled type’s _actual_ constructor)
    *   This has linear (_O(n)_) complexity. For a constant-time allocation, use a freelist, like this:
        
            class Particle {  
            public:  
              // init(), render(), etc.
            
              Particle * get_next() const { return m_state.next; }  
              void set_next(Particle * next) { m_state.next = next; }
            
            private:  
              int m_frames_left;  
              union  
              {
                // State when it's in use (m_frames_left > 0)  
                struct  
                {
                  double x, y;  
                  double x_vel, y_vel;  
                } live;
            
                // State when it's available (m_frames_left == 0)  
                Particle * next;  
              } m_state;  
            
        
        *   Using a union allows us to give the _same_ memory a _different_ semantic interpretation when the particle isn’t in use—no need to keep a separate chunk of memory around to implement the freelist.
            *   We’re cannibalizing the memory of the unused particles themselves!
        *   Then, the pool just keeps a pointer to the first available `Particle`, and when one goes inactive, it becomes the new head of the list.
*   Design decisions
    *   Are objects coupled to the pool?
        *   If yes, the implementation is simpler–the pooled type can store an “in use” flag or implement a Boolean `in_use()` method. You can also ensure that only the pool is able to create the objects.
        *   If not, the objects can be of any type, but you have to track the in-use state separately.
    *   Who is responsible for initializing the reused objects?
        *   If the pool does it, you can completely encapsulate the objects (ensuring no external code maintains references to objects that could be reused without warning), but the pool is tied to the objects’ initialization code—might need many `init()` functions on the pool to pass arguments through to the pooled object’s `init()`.
        *   If outside code initializes them (i.e., the pool just returns a pointer to the “new” object), the pool’s interface gets simpler, but outside code has to cope with the possibility of an allocation failure.
*   See also: packing a bunch of the same object together in memory improves performance as you iterate them, per [the Data Locality pattern](#chapter17-datalocalityi.e.data-orienteddesign) (Chapter 17).

### Chapter 20: The Spatial Partition Pattern

> Efficiently locate objects by storing them in a data structure organized by their positions.

*   Motivation: Suppose you have a number of military units on a field of battle, and each frame, each unit has to decide which _other_ unit(s) to attack. The naive implementation is _O(n2)_, as each unit iterates through all other units to choose the ones in the same position as themselves.
    *   This too slow for large numbers of units!
    *   Idea: Imagine we had a 1-dimensional battlefield; each unit could be stored in an array, and its position within the array would correspond to its location on the battlefield. Then, finding the nearest units becomes easy!
*   Solution: Store each object in a spatial data structure that organizes the objects by their location. Then, you can efficiently query for objects at or near a given location. When an object’s position changes, you’ll have to update the data structure so that it can continue to find the object.
    *   Works for both static objects (like 3-D clutter, whose positions can be determined once at initialization) and for moving objects
        *   But, implementation for moving objects is significantly more complex (imagine a hash table whose keys can change dynamically based on changes to the object!) and requires extra computation.
    *   Obviously becomes more valuable as _n_ increases; for small _n_, don’t bother!
    *   Like other perf tricks, this trades memory (in the form of the data structure) for time.
*   A simple implementation
    *   The 2-D space is represented by a `Grid`, which contains a number of cells.
    *   Each cell stores a pointer to a linked list of units.
    *   A unit is added like this:
        
            void Grid::add(Unit* unit)  
            {
              // Determine which grid cell the unit is in.  
              int cell_x = (int)(unit->x / Grid::CELL_SIZE);  
              int cell_y = (int)(unit->y / Grid::CELL_SIZE);
            
              // Add to the front of list for the cell it's in.  
              unit->prev = NULL;  
              unit->next = m_cells[cell_x][cell_y];  
              m_cells[cell_x][cell_y] = unit;
            
              if (unit->next != NULL)  
              {
                unit->next->prev = unit;  
              }
            }  
            
        
    *   Then, to handle combat, we walk each cell in the `Grid` and call:
        
            void Grid::handle_cell(Unit* unit)  
            {
              while (unit != NULL)  
              {
                Unit* other = unit->next;  
                while (other != NULL)  
                {
                  if (unit->x == other->x &&  
                      unit->y == other->y)  
                  {
                    handleAttack(unit, other);  
                  }
                  other = other->next;  
                }
                unit = unit->next;  
              }
            }  
            
        
        *   This looks a lot like the original code, _but_ `handle_cell()` ends up walking drastically fewer nodes—instead of looking at all the units across the whole battlefield, we’re _only_ considering the ones in the same cell.
            *   This depends, of course, on the granularity of the cells—if they’re _too_ granular (they contain too many units at once), we’re back where we started.
    *   Of course, as a unit moves, it needs to update its position in the grid
        
            void Grid::move(Unit* unit, double x, double y)  
            {
              // See which cell it was in.  
              int old_cell_x = (int)(unit->x / Grid::CELL_SIZE);  
              int old_cell_y = (int)(unit->y / Grid::CELL_SIZE);
            
              // See which cell it's moving to.  
              int cell_x = (int)(x / Grid::CELL_SIZE);  
              int cell_y = (int)(y / Grid::CELL_SIZE);
            
              unit->x = x;  
              unit->y = y;
            
              // If it didn't change cells, we're done.  
              if(old_cell_x == cell_x && old_cell_y == cell_y) return;
            
              // Unlink it from the list of its old cell.  
              if(unit->prev != NULL)  
              {
                unit->prev->next_ = unit->next;  
              }
            
              if(unit->next != NULL)  
              {
                unit->next->prev_ = unit->prev;  
              }
            
              // If it's the head of a list, remove it.  
              if(m_cells[old_cell_x][old_cell_y] == unit)  
              {
                m_cells[old_cell_x][old_cell_y] = unit->next;  
              }
            
              // Add it back to the grid at its new cell.  
              add(unit);  
            }
            
        
        *   Essentially: if the unit has changed cells, remove it from its old place in the list, and re-insert it at its new location.
            *   Doubly linked list allows us to update the data structure efficiently (just change a few pointers).
    *   What if you want the units to attack any other within a range (rather than in the exact same location)?
        *   Consider the case where two units are just across cell boundaries: Unit A is at the rightmost edge of one cell and Unit B is at the leftmost edge of the adjacent one.
        *   Need to compare units in adjacent cells… but don’t bother checking _every_ cell around _every_ unit, because you’ll end up doing twice as many comparisons as necessary. (In essence, you would be checking if Unit A is within distance of Unit B, then if B is within distance of A.)
        *   Instead, for each unit in a given cell, you want to check the lower (in terms of iteration order) adjacent cells. If your unit is in the cell marked U, you would check the cells marked with an X:  
            
              
            
            X
            
            X
            
            X
            
            U
            
            X
            
*   Design decisions
    *   The following discusses specific types of spatial partitions, organized by their essential characteristics—choose a specific data structure based on how & why they work, rather than, um, it being the only spatial partion you’re familiar with.
    *   Is the partition hierarchical or flat?
        *   Background:
            *   Our grid example uses a single flat set of cells
            *   Alternatively, hierarchical spatial partitions divide the topmost level of the space into just a few regions; if any of those regions contain more than a few objects, they subdivide the region again, and so on until every region has fewer than a set max number of elements.
        *   If you choose a flat partition, it’s simpler, memory usage is constant, and it can be faster to update when elements move.
        *   If you choose a hierarchical partition, it handles empty space more memory efficiently (no big set of empty cells), and it handles densely populated areas more computationally efficiently—whereas a flat partition might end up (through bad luck) with all its units in a single cell (putting you back to _O(n2)_ performance), a hierarchical partition guarantees you only a few units in a given cell.
    *   Does the partitioning depend on the set of objects, or are the partition boundaries fixed in advance?
        *   If the partioning is object-independent:
            *   Pros:
                *   Objects can be added incrementally without performance issues (you won’t have to rethink your entire partitioning scheme!)
                *   Objects can be moved quickly—again, you won’t ever wind up having to re-sort and re-balance a bunch of nodes
            *   Con: partitions can be imbalanced, and you can wind up with too many elements in a single node.
        *   If the partioning adapts to the set of objects it’s storing
            *   E.g.:
                *   k-d trees and BSPs, which recursively divide the space in half so that each half contains about the same number of objects
                *   bounding volume hierarchies
            *   Pro: partitions are balanced, so you’re guaranteed _consistent_ performance
            *   Con: It’s more efficient to partition an entire set of objects at once, since adding objects causes you to shuffle things around. (For this reason, these are best for static objects that won’t move throughout the game.)
        *   If the partitioning is object-_in_dependent, but the hierarchy is object-_dependent_
            *   E.g., quadtrees (have some of the best characteristics of both fixed partitions and adaptable ones)
                *   Begins with the entire space as a single partition
                *   If the number of objects in the partition exceeds a threshold, it is subdivided into four smaller squares, the boundaries of which are always fixed—they always divide in half.
                *   Since we only subdivide squares with too many elements, the partioning adapts to the set of objects, but the partition bounds are fixed.
            *   Pros:
                *   Objects can be added incrementally (since the number of objects you’ll have to move within the data structure has will be the predefined max for a cell)
                *   Objects can be moved quickly (since a “move” is just a “remove-then-add”)
                *   The partitions are balanced
        *   Are objects stored _only_ in the partion, or is the partion a secondary cache?
            *   If the partition is the only place the objects are stored, you save memory overhead and the complexity of managing two collections.
            *   If another collection actually stores the objects, traversing all the objects is faster—you have two data structures, each optimized for its use case.
*   Varieties of spatial partitions (for further reading), along with the 1-D data structure on which they’re based
    *   Grid (a persistent [bucket sort](https://en.wikipedia.org/wiki/Bucket_sort))
    *   Types of binary search trees:
        *   Binary space partitition (BSP)
        *   k-d tree
        *   Bounding volume hierarchy
    *   Quadtree (and its 3-D version, octree) are [tries](https://en.wikipedia.org/wiki/Trie)
