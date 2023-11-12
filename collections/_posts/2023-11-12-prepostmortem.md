---
title: "A Prepostmortem: Why SleepEasy Will Fail"
layout: post
authors: ['tyler']
categories: ["Work", "Entrepreneurship", "Build in Public"]
excerpt_separator: <!--more-->
---

There's an exercise for founders that strikes me as really valuable:

> Suppose it's a year from now and the business has failed. 
> 
> _Why_ did it fail?

This is surprisingly insightful. In the case of [SleepEasy](https://www.sleepeasy.app), the biggest risks are not technical... after all, I'm basically building a Cadillac version of website monitoring. There are certainly *some* interesting technical challenges (we'll get to those), but it's not like I'm trying to build a wearable lapel pin to replace the smartphone. 👀

<!--more-->

## Market risks

No, the biggest risk—the #1 reason I think the business would have failed in this hypothetical future—is simply an inability to gain customers quickly enough. This has a few angles:

First, I don't even know who my target market really is. I'm reasonably well connected to a lot of web developers, and it's certainly the case that SaaS companies need monitoring. However, those teams are often quite sophisticated, and they almost certainly have at least _some_ overlapping tech already in place. Moreover, anyone with a dev team is likely to look at SleepEasy and say to themselves "we could cobble that together ourselves from our existing tools _x_, _y_, and _z_." In my experience that's _true_ in the strictest sense, but it's also not realistic. What actually happens is that this sort of monitoring continually gets pushed down the priority queue; while these teams _could_ build it themselves, they'll never get around to it because they're overbooked on feature work.

Instead, I'm starting to wonder whether a better customer base might be less technical teams, like operators of ecommerce sites. People who build on, say, WordPress and WooCommerce are quite comfortable paying for software that solves their business problems, and while they might have a developer or two around (maybe on contract), they generally don't have so much in-house technical expertise that they'd consider building sophisticated monitoring tools themselves. (The other advantage of going after WordPress sites, of course, is built-in distribution via the plugin marketplace.) Taking this route is essentially an attempt to bring the benefits of observability to folks that have never heard the word observability. 😄

Perhaps even more scary than not getting the ideal customer profile right: maybe my would-be customers are simply apathetic. Maybe downtime or feature outages simply aren't a problem for them, or not a _burning_ problem bad enough that they're actually motivated to solve it. Maybe being in the WordPress plugin directory doesn't help because no one is even searching for monitoring. (This one strikes me as less risky—clearly there's a market for other, downmarket website monitoring services. But it's certainly still a possible cause for growth being too slow to make the business sustainable.)

The final concern I have on the market side: maybe the decisionmakers here are CTOs or other highly ranked, busy people in a company that I simply can't reach. Maybe the people who really would benefit from SleepEasy (like the on-call folks who are no longer being woken up at 2 am by false alarms) don't have the organizational pull to actually make purchasing decisions.

## Technical risks

Those market risks are certainly the ones that concern me the most, but there are technical and staffing challenges that might kill the product as well.

The biggest is with this whole idea of having a human confirm an alert before it goes to customers. There are actually three aspects to this:

1. If I can't staff this appropriately with people in other timezones, I'll have made my own life miserable as my family time, sleep, and of course productive work hours are interrupted constantly as I review alerts.
2. I'll need the signal-to-noise ratio on those alerts to be extremely high. There's obviously a lot of incentive to make the checks reliable, so that my team won't be bogged down with false alarms.
3. Maybe customers just legitimately have a _lot_ of incidents—that is, the noisiness of the alerts (and excessive load on my team) might come not from false alarms, but from real issues.

The last technical risk is around the features I'm not aware of other companies building in this space: what if I'm simply not able to automate the fancier checks that I need (authentication, billing, visual regression testing, etc.), and I'm unable to charge enough for those services to pay humans to do it?

## Okay, let's not step on those landmines

Thinking through these things is valuable. I feel like it's basically a map of where the mines are buried along the path ahead. There may yet be other mines I can't see in advance, but at least I can address some of the danger proactively.

As always, I'm interested to hear any feedback you might have on this stuff. Are there other risks I've not captured here? If you'd like to think through your own prepostmortem, I'm happy to be a sounding board—you can find me using the social links below.
