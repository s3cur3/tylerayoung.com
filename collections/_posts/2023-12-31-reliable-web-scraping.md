---
title: "Making Web Scraping Super Reliable"
layout: post
authors: ['tyler']
categories: ["Entrepreneurship", "Build in Public"]
excerpt_separator: <!--more-->
image: /assets/images/globe-night.jpg
image_credit: Photo by NASA
---

Two of the core features of SleepEasy—uptime checks and broken link checking—are built on web scraping. In this last half of December, as I gear up for a big marketing push and public signups, I've been doing a lot of work to get these features really dialed in and reliable.

<!--more-->

The naive approach to web scraping is so simple! You make an HTTP GET request and... bam! Done! As with everything in software engineering, though, things are way more complicated than they seem.

## Challenge 1: Don't overwhelm the host

First, SleepEasy needs to be respectful. We don't want to overwhelm a site and inadvertently cause a denial-of-service for real users. For this reason, we're super conservative: for each domain we're crawling, we only ever imitate a single browser request at a time. While Elixir makes it easy to parallelize work, it would be disastrous for some customers' sites if we ran thousands of requests in parallel. Instead, my goal was to ensure the load placed on the site was no worse than that of a single human with a particularly short attention span.

That doesn't mean we only run a single HTTP request at a time, though. Real browsers will request the initial page _plus_ a ton of static assets all at once. We do the same—we'll load CSS, JavaScript, and image files in parallel with the HTML of a page.

## Challenge 2: Don't get the requests blocked

The next category of challenges we face come in the form of anti-bot measures. A great many sites have a service like Cloudflare in front of them as a way to prevent DDoS attacks, and despite trying to be respectful of the server's resources, these anti-bot measures will only give you an HTTP 403 with the naive approach.

The solution is conceptually simple: we need the requests our scraper makes to be indistinguishable from a real human. Tools like Selenium and Chromedriver allow us to automate control of an honest-to-God full browser, which works swimmingly for getting around anti-bot measures. They have their own issues, though...

## Challenge 3: Don't rely on real web browsers any more than we have to

There are two huge problems with running a real browser for every request:

1. Memory use. It's easy to encounter pages that will cause Chromedriver to allocate 6 GB of memory or more.
2. Reliability. Chromedriver is a huge, complicated piece of software, and when it breaks, it does so in ways that will be utterly inscrutable to you from the outside. The "solution" is generally to shut down the instance and boot a new one, but that wastes time _and_ requires making additional requests to the site you're scraping... and it assumes you're first _able_ to detect that the problem is with Chromedriver and not the website you're checking.

A smaller problem—but a problem nonetheless—is speed. It's just way slower to scrape a page in Chromedriver compared to making a plain HTTP request and parsing what you need yourself.

For all these reasons, I've spent a great deal of time getting my non-browser-based scraping to imitate a browser's request lifecycle as closely as possible. That starts with faking the user agent, but goes way farther. The end result is that it lets me make requests to Cloudflare-protected sites (and sites like with notoriously stringent anti-bot measures like LinkedIn) without running an actual browser. I get better speed, _much_ better reliability, and a tiny fraction of the memory use of a browser for the overwhelming majority of cases, and I can fall back to Chromedriver only as a last resort, or on sites that are fully client-side rendered and therefore actually to execute JavaScript.

## Challenge 4: Don't get your IP blocked

Even with the best scraping tech around—including just running Chrome!—if you scrape too much from a single IP address, Cloudflare denial-of-service protection will kick in and block your IP. As best I can tell, this is what's happened for the whole IP range of GitHub Actions—end-to-end tests that work 100% of the time on my machine will instead be blocked 100% of the time if I try to run them in CI. The trick, then, is to keep the amount of scraping coming from your IP to a low enough level that it escapes Cloudflare's notice. This is kind of a reiteration of the "be respectful" point I started with, but extended to include "noisy neighbors" in a shared hosting environment—they'll ruin the party for everybody! Since I'm [running my own server](https://x.com/TylerAYoung/status/1730253716073148470), this isn't something I've had to deal with, but it'd be a concern on many hosts.

## Is this commercially viable on its own?

I'm toying with the idea that this web scraping tech might be worth selling on its own—"selling your by-products," [as Jason Fried called it](https://signalvnoise.com/posts/1620-sell-your-by-products) years ago. It's the sort of thing where I'd want a small customer base—in part because I'd want to make sure every person I sold it to was going to use it respectfully, but also because if I sold it to a thousand companies, the techniques used would get picked up by anti-bot systems and be blocked.

The absolute last thing I would want to do is to operate a scraping _service_. My own experience with these services is that they're horribly unreliable—about as unreliable as Chromedriver is when I'm running it locally myself! I've tried a handful, and I frequently get spurious timeouts and errors. Plus, they're fundamentally slower as a client compared to running the scraping yourself (there's an extra network hop in there, of course)... not to mention rotating IP addresses is the absolute last thing I want to be on the hook for.

Instead, I'm wondering if I could sell this as a paid library, similar to the Oban model. For, I dunno, $100/month, you get access to a library that will do the scraping for your app with excellent reliability, the absolute minimum response times, and rounds-to-zero additional server load. No additional microservices, no third-party APIs, just the scraping.

That certainly would have been a good deal at companies I've worked for. I've spent a developer-week (to the tune of $2,000–$6,000) multiple times in my career to put together a much _worse_ solution that we just had to live with, because we couldn't afford to have me spend as much time as it would take to make it really good. And the fact that scraping is a constant cat-and-mouse game means it requires ongoing maintenance to keep things working. I'm in a good position to do that, given that it's core to my business, but product companies really aren't.

If that's something that would be interesting for your team, give me a shout.
