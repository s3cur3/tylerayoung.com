---
title: "Acquisition Channel Determines Pricing (And Product)"
layout: post
authors: ['tyler']
categories: ["Entrepreneurship", "Build in Public"]
excerpt_separator: <!--more-->
image: /assets/images/colorado-2017/lone-tree-hdr-banner.jpg
---

<iframe
 width="207"
 height="368"
 src="https://www.youtube.com/embed/1-iigCRrHxk?autoplay=1&loop=1&rel=0"
 title="SleepEasy demo video"
 style="float: right; margin-left: 2rem; margin-bottom: 0.5rem;"
 frameborder="0"
 allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
 allowfullscreen>
</iframe>

This week, I [launched](https://x.com/TylerAYoung/status/1742944121663139877) a demo feature for SleepEasy to make it really easy for prospective users to see what the app can do for their site. Right at the top of [the home page](https://www.sleepeasy.app), you just drop in your site's URL and we'll check:

- Uptime (is your site working at all?)
- Broken links (we'll try to load the first 5,000 links, images, scripts, etc.)
- HTTPS support (to make sure your SSL/TLS certificates aren't going to expire and thereby break access to your site)
- Domain registration (to make sure a missed renewal doesn't cause you to lose access to your domain)

Through social media, I got about 30 people to try their site—not a bad amount of traffic, I thought, but only _one_ of those 30 actually created an account (a _free_ account!). That is, only one person cared enough about this stuff to sign up for alerts when we found issues. Of course, I already knew [the people in my network are not my target market](/2023/12/09/early-access-launch/), but this was just further evidence.

So I need to shift my focus fully to marketing, and specifically to traffic acquisition. I need to get SleepEasy in front of people who:

- Know they have a need for website monitoring
- Know they lose money when there are issues with their site
- Have had issues with site reliability in the past

I have ideas on where to find people like that, but they aren't going to be the people I originally intended to sell SleepEasy to. Let me explain...

<!--more-->

## Don't call it a pivot

When I started SleepEasy, it was with the intention of satisfying a need that SaaS companies have for real-user testing on their site. I'm not abandoning that idea entirely, but after 50 attempts at cold outreach to technical folks at SaaS companies, plus a dozen or so warm contacts within my professional network, my conclusion is that this is a new market, and new markets are notoriously hard to bootstrap. Companies might have a need here, but it's not something they're actively looking for (they don't know it exists), and they certainly don't already have budget allocated for it.

To that end, I had hoped to use uptime monitoring (a decidedly established category) as a means of getting a foothold in a company, so that I could later sell them on more advanced monitoring services. This, too, has simply not worked. I've not seen _any_ interest from the people in the product now—they're quite happy with the existing (free) check types.

As Jason Cohen optimistically observed in his famous talk "[Designing the Ideal Bootstrapped Business](https://www.youtube.com/watch?v=otbnC2zE2rw)," you can see it as a gift when the business isn't working. It means you can try new things without fear of loss. What's the worst that could happen?

For that reason, I'm doubling down on the things that _are_ sort of working, and dropping the things that absolutely aren't. That means focusing on being a really, really good service for "uptime monitoring, but better."

## Okay, but who's gonna pay for this?

"Uptime monitoring, but better" is not really a service SaaS companies (my original target market) are looking for. It's not that they don't need it, but rather they're almost always paying for other services that bundle uptime monitoring in for free. Why pay for SleepEasy when Datadog/New Relic/Pingdom/AppSignal/Sentry/etc. can already do that? (Never mind the benefits SleepEasy has over the competition, like broken link checking or the "zero false alarm" feature—it's a *lot* of work to get someone to switch from an existing service to a new one unless they're extremely disgruntled. Inertia's hard to overcome.)

So who _is_ buying uptime monitoring as a standalone service—people who might really be persuaded to move upmarket (if only just a bit) and buy SleepEasy? Well... how about WordPress site operators, people running ecommerce sites, affiliate blogs, etc.? They fit the criteria I listed above:

- If they've been around long enough, they've had issues with reliability—a plugin update broke the site in a way they didn't notice, perhaps, or they got a spike of traffic that brought the site down. (Ah, WordPress!)
- Uptime is money to them in a very direct way.
- They frequently do _not_ have other observability tooling that bundles uptime monitoring in for free.

The problem I'm running into is that my service as originally conceived was _extremely_ upmarket. Human review of every alert is expensive, and while people operating a business on WordPress are used to spending money on features for their site, $29/mo is a hard sell for all but the largest players in the space. I'm going to lose out on price alone, before they even look at what they get for that price.

So, while the WordPress plugin directory is a huge source of potential traffic, it's not traffic I can monetize at my original price points.

## WordPress as the primary acquisition channel caps the price

If I'm going to be in the WordPress plugin directory, there are a few big changes I have to make to the pricing strategy:

- Freemium is a must. The plugin has to do _something_ when you install it. Similar to the demo, my plan is to sharply limit the number of issues you can see, as well as limit the frequency of uptime checks for free users. Those are built-in opportunities to sell them on a paid plan.
- I need a cheaper entry plan. Something that's at least in the ballpark of my $0–5/mo competitors like Pingdom or UptimeRobot.
- The $99+/mo plans need to go. That might be a conversation I have with individual customers who sign up for the highest tier, but it's not something I should devote space to on the home page.

With that in mind, I'm thinking the new tiers will be something like free, $7, and $27. (I've heard prices that end in a 7 are all the rage...) Only the highest tier will have the "zero false alarms" guarantee—I'm not willing to be woken up at 3 am for $7/mo. And only the highest tier will get the highest-value checks I have planned—things like testing contact forms, email deliverability, etc.

## WordPress as the acquisition channel sets the product roadmap

Focusing fully on WordPress also gives me a clarity around what features I should build going forward. I can build integrations specifically with WooCommerce (the _de facto_ ecommerce plugin for WordPress), and Contact Form 7 (the _de facto_ plugin for creating contact forms), etc.

The price point, too, informs the product direction. If the most expensive tier is $27/mo, I can abolish the notion of any features that regularly require more than a moment of a human's time.

## Should I consider this a sunk cost and move on?

Would I have gone into building this product knowing what I know now about the market? Ehhhhh... probably not. The received wisdom from the bootstrapped SaaS community is that neither freemium nor low price points are conducive to building a business. There are a few reasons I'm not deterred, though.

First, freemium in the WordPress market specifically will cost me almost nothing. Because 99%+ of WordPress sites are fully server-rendered, they're amenable to [crawling without needing a browser](/2023/12/31/reliable-web-scraping/). Crawling a client-side-rendered site is hugely expensive for me (a single page might eat 6 GB of RAM!), but server-rendered pages require so few resources that they're basically free.

Second, I don't anticipate support load scaling much with the number of free/low price point users on the platform. Because there is literally _zero_ configuration at the low end, it's very likely that any issue a user encounters represents a bug in the core platform—something I'd like to find out about anyway so I can fix it. That's in stark contrast to a business where new users need a lot of handholding to get started, and they might need ongoing support across the lifetime of their use of the product.

Third, I'm not in a hurry. If it takes 1200 customers to reach my goal of $8k/month, and it takes 3 years to acquire that many customers, so be it. I can keep contracting part-time to pay the bills, and I'm quite happy working on this business in particular. (Much happier than I'd be trying to build something for, I dunno, countertop installers.)

## Thoughts?

Am I making a big mistake? Have I overlooked something crucial? I'd love it if you'd reach out on [Twitter](https://twitter.com/TylerAYoung), [Mastodon](https://fosstodon.org/@tylerayoung), or email (my first name @ this domain).
