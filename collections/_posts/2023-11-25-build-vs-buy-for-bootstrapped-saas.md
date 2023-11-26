---
title: "Build vs. Buy for a Bootstrapped SaaS"
layout: post
authors: ['tyler']
categories: ["Work", "Entrepreneurship", "Build in Public"]
excerpt_separator: <!--more-->
---

Since I made the [decision](/2023/10/30/going-indie-building-in-public/) to go independent and build SleepEasy, I've faced a couple points where I needed to decide whether to build a particular feature from scratch myself or buy someone else's code to do the job.

In my years of working at small software shops, the common refrain was that we would consider buying tech to solve a problem if it wasn't core to our business or part of our unique selling proposition. I don't really feel like that principle applies at the MVP stage of SleepEasy, though. As I'll discuss below, I happily considered shelling out to a third party for a core feature of my app, just because the value to customers is in the way that feature gets wired up to their web site. At this stage of the product, having *something* to fill a gap is better than spending months trying to make the best, most unique version of a feature.

Unsurprisingly, cost is also a much bigger factor for me than it was at my Real Jobs™. When you're paying a developer, say, $15k/month, it's easy to justify spending thousands of dollars buying a solution if it lets you use that developer's time for something else. In my case, of course, SleepEasy's current lifetime revenues amount to $0, so that calculus doesn't quite work the same. At the same time, though, there's an opportunity cost to me not having a Real Job. Every month I'm trying to bootstrap SleepEasy is a month my family is foregoing tens of thousands of dollars, so if the business is going to fail, it's better to do so quickly. The right amount for me to be willing to spend, then, is not $15k per month of dev time saved, but it's also quite a bit higher than $0.

So that's where my head has been on these things. I have three examples I can share for how those ideas have worked out.

<!--more-->

## Build vs. buy a starting point SaaS app

This one was easy. I'm a huge fan of [Phoenix](https://www.phoenixframework.org) and all the things you get with it out of the box, but one of their mottos is "Phoenix is not your app." Fair enough—it's the web layer. In the past, I've built the infrastructure that goes around the other core SaaS concepts, like admin panels, UI components, organizations and multi-tenancy, emailing, auth (including Oauth!), internationalization, etc. etc. Because I've built that stuff, though, I know how incredibly time consuming it can be. When I ran across the [Petal Pro framework](https://petal.build), and they offered all that stuff pre-built with very solid code (better than when I did it at past jobs!) for $300, it was an instant sale. If you're starting a fullstack Elixir project, you'd be kind of crazy not to buy this or something like it.

- **Time saved**: 40-80 hours for the pieces that would have been essential to me, plus God only knows how many hours for the stuff I'll eventually use
- **Cost per hour saved in the near-term**: $3.75&ndash;7.50
- **Conclusion**: Massive, ridiculous, outsized value

## Build vs. buy really nice web scraping

Web scraping—that is, my app being able to fetch and interpret a web page on a customer's site—is at the core of two of SleepEasy's main value propositions: uptime checks (to make sure the customer's home page is serving something reasonable) and scanning for broken links. 

Unfortunately, scraping in the modern day is kind of notoriously difficult. There's a perpetual game of cat-and-mouse going on between malicious or ignorant web scrapers ("bots") who put too much load on a company's web servers and the bot detection tools that try to block them. Carefully imitating a real web browser gets you a long way, but I kept running into instances where I was getting blocked when checking my _own_ sites at 30 second intervals. Netlify, where I host all my static sites, has some serious bot detection going, which turns out to make a great test case for how my scraper is doing in the wild, but it also meant I was getting a false alarm every 10-15 minutes. It was clear I'd need to up my game to get my web scraper to production.

To that end, I thought it might be worth it to fall back to a third-party service to confirm a page was unavailable before I consider it down. I was hoping this would be an accelerator on my productivity, because I could easily imagine it taking weeks for me to get my own scraper up to snuff, and I _really_ want to hit that December 1 date for my early access launch. I wired up Apify, one of the more transparently priced services of this breed, and set it up as a fallback in case my own checking failed. However, at roughly $0.01 per page scraped, it still wasn't going to be sustainable—for sites like my own, where I was falling back to Apify frequently, it could easily cost me $15/month or more per site. (When checking at 30 second intervals, it doesn't take a high failure rate for those pennies to add up!) Worse yet, it didn't solve my reliability issue—maybe 10-15% of the time when I asked them to check a page, they *too* would have an issue leading to a false alarm.

In the end, I spent the last week doing the very thing I had tried to avoid: improving the reliability of SleepEasy's own web scraping. That work has paid off, though, and I've now been running the app more than 48 hours now without any false alarms, and without falling back to Apify. In the future, Apify will still be useful for doing multi-region checking to confirm downtime (I can ask them to scrape a page from, say, 4 continents, and only alert if all 4 fail). For the time being, though, the integration turned out to be a waste of time. This is probably part of the wisdom of saying you shouldn't try to buy the tech if it's core to the business—my standards for reliability here are way higher due to the frequency with which I need the uptime checks.

- **Time lost**: ~4 hours for an integration that wound up being a dead end (for now)
- **Cost**: $0 so far (at least there's that...)
- **Conclusion**: Seemed like a good idea, but didn't work out in practice. Scraping is core to my business, and it's both too expensive and too unreliable to delegate to a third party.

## Build vs. buy someone else's entire codebase

A surprising bit of serendipity occurred during the first week of me building in public. I posted about going independent, and about what SleepEasy was doing, and someone I really respect from Twitter reached out offering to share their experience from trying a similar business. Obviously I jumped at the chance to hear what had worked and what challenges they faced, and in the course of the conversation, I learned:

- They had built their service in Elixir (💜)
- The architecture of their codebase was very similar to my own
- They had built out two of the checks (domain name registration and SSL/TLS certificate checking) that had been on my list for post-early access, but which I knew I wouldn't be able to prioritize for this year
- They had also built out a bunch of other ancillary features like Stripe integration that I knew I'd eventually need as well
- A little over a year ago, they took a high-stress job as a CTO and shut the side business down

As these pieces kind of clicked together, I eventually had to ask: "So, uh... would you be interested in selling me your codebase?"

I don't think the thought had occurred to the person in advance, but they were open to it. I figured it would:

- save me between two to four weeks of (part-time) development on the checks themselves,
- save me that amount again on the Stripe integration I'd need sometime in the next 6 months, _and_ 
- make the early access offering quite a bit more compelling.

I couldn't offer what their time was really worth (even if it had only taken them 80 hours at $100/hr, that was way more than I could put together at this stage of the business). But I tried to come up with a number that wouldn't be _insultingly_ low, something I'd be happy to be on the receiving end of, so I offered them $3,000. Happily, they accepted, and we completed the deal with a virtual handshake over Twitter DMs.

- **Time saved**: 40+ hours in the next 3-4 months, 80+ hours in the next 6 months
- **Cost per hour saved**: $37.50&ndash;75
- **Conclusion**: Bespoke software is expensive (and feels ever moreso being on the buying end), but in this case, the acceleration of my own timeline feels very worth it for me. I was able to drop in those two checks in a couple hours instead of a couple weeks. As an added bonus, after integrating their code and using my own for awhile longer, I've concluded the architecture of my own job runner was ultimately problematic in ways I hadn't foreseen (specifically with resuming jobs between restarts of the app), while theirs works around the exact limitations I was facing. That probably saved me another 10 hours of R&D.

## No easy formula

In the end, I'm finding this stuff is messy. I don't have a conclusive formula for how to make these decisions, and my gut hasn't been right every time. The one thing I'm very confident of is not valuing my time at $0. It hurts to spend money on things at this stage of the business, but I'm trying to remind myself that the opportunity cost of not having a "real" job exists, and vastly outweighs this stuff. If it's essential, or really is guaranteed to save me lots of time and get to market (or fail!) faster, that's a good tradeoff.


