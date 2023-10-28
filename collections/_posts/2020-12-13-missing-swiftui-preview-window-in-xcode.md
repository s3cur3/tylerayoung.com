---
title: "Missing SwiftUI Preview Window in Xcode"
layout: post
authors: ['tyler']
categories: ["Programming", "Apple Development (Swift, SwiftUI, Objective-C, AppKit, and UIKit)"]
---

This is no doubt obvious if you’ve dutifully watched all the WWDC sessions on SwiftUI, but if you’re like me and just diving in and hacking the hell out of it, you might be baffled that the much-renowned preview pane isn’t showing up when you create your SwiftUI `View`.

There are two things you need:

1.  The Canvas view enabled in the upper right (also available via Cmd+Option+Enter)
    ![Enabling the canvas view in the upper right of Xcode 12](/assets/images/show-canvas-view.png)
2.  A `PreviewProvider` declared that uses your new `View`. This is what the preview pane picks up automatically for rendering—_just_ having the view, even if it has default-assigned data to display, isn’t enough. Here’s what that might look like:

```swift
import SwiftUI

struct LibraryView: View {
    let paths: [LibraryDirectory] = [LibraryDirectory(fromPath: "~/Dropbox"), LibraryDirectory(fromPath: "~/Documents")]

    var body: some View {
        List(paths) { paths in
            Text(path.toString())
        }
        .frame(minWidth: 300, minHeight: 300)
    }
}

// The important bit!
struct LibraryViewPreview: PreviewProvider {
    static var previews: some View {
        LibraryPicker()
    }
}
```





