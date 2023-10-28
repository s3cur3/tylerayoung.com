---
title: Iterating Over a Generic Sequence in Swift
layout: post
authors: ['tyler']
categories: ["Programming", "Apple Development (Swift, SwiftUI, Objective-C, AppKit, and UIKit)"]
---

Here’s a goofy bit of generic programming I do all the time in Swift—ironically, it’s one thing C++ makes easier than Swift!

The use case is simple: I want a function to be able to accept a container (really a Swift [`Sequence`](https://developer.apple.com/documentation/swift/sequence)) of any type, as long as its _values_ are of the right type. For instance, I want it to be able to take a `Set<String>`, an `Array<String>`, or a `Whatever<String>`… I don’t care what the container type is.

This is conceptually easy enough (“I want a `Sequence` whose whose elements are `String`s), but I have to look up the syntax every time:

```swift
func countChars<Seq>(someStrings: Seq) -> [Int]
    where Seq: Sequence, Seq.Element == String {
  return someStrings.map({ $0.count })
}

let strings = Set(["1", "22", "333"])
let chars = countChars(someStrings: strings)
print(chars) // > some ordering of [1, 2, 3]
```

And here’s what it looks like when writing an initializer instead of a normal function:

```swift
extension Dictionary {
  init<Seq>(mapping seq: Seq, to transformer: @escaping (Seq.Element) -> Value)
      where Seq: Sequence, Seq.Element == Key {
    self.init(uniqueKeysWithValues: seq.lazy.map { ($0, transformer($0)) })
  }
}

let strings = Set(["1", "22", "333"])
let charCounts = Dictionary(mapping: strings, to: { $0.count })
print(charCounts) // some ordering of ["1": 1, "22": 2, "333": 3]
```

[Here’s a demo of the above examples](https://repl.it/@s3cur3/VastAlarmedDecagon) you can play around with.


