---
title: "The Worst Bug I Ever (Nearly) Shipped"
layout: post
authors: ['tyler']
categories: ["Programming", "Work"]
---

The year was 2014. I was wrapping up about a year’s worth of work on the X-Plane 10 Mobile release, and we were all set for a Christmas release. Timing this was difficult—we were pressed for time, and the App Store approval process takes an _indeterminate_ amount of time; you submit a binary, then wait (in those days, at least 7, sometimes more than 14 days) and hope they approve it. If they don’t—if they reject the app for any reason—you have to go fix the issues they identified, then submit a new binary (and go to the back of the line).

To make matters worse, the App Store approval process _shuts down entirely_ for about a week around Christmas. So, if we missed the December 22 deadline, we wouldn’t get the app out at all until the first of the year.

So, on December 16th, Chris (the project manager for the mobile app) called me. “Congrats,” he said. “We made it! They’ve approved us, and I’m going to go do a little bit more testing and then mark the app for sale.”

“Awesome!” I said. “I’ll do a little more testing as well.”

…

You can tell where this is going.

All the aircraft in X-Plane Mobile are sold separately, somewhere between $0.99 and $4.99. But, we offered two ways to try the paid planes for free: a 60 second trial flight, or a free 24-hour “unlock” with one catch: you had to post something on Facebook or Twitter about X-Plane in order to get the 24 hours of playtime. This whole “pay-with-a-post” thing was my brainchild… I had read about other apps doing it, and I had been 100% responsible for the implementation.

In my testing, I discovered that sharing to Facebook worked fine… but the app wasn’t actually unlocking the planes after you did so!

_I had inadvertently introduced the most infuriating bait-and-switch imaginable._

I was nearly ill as I called Chris. I explained what happened—how my screwup was going to force us to have to pull the app, resubmit it, and entirely miss the Christmas release window. To his credit, Chris was devastated, but he didn’t berate me about it. Bugs happen, he said, so let’s just get it fixed ASAP and move on.

I got the one-line fix committed as soon as we got off the phone. Chris was able to contact Apple and explain the situation. As luck would have it, they took pity on us and went ahead and reviewed the fixed version of the app (without sending us to the back of the line). We got the app out the door the next day, in time for Christmas break, and we both breathed a huge sigh of relief.





