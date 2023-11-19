---
title: "On MVPs and Cutting Scope"
layout: post
authors: ['tyler']
categories: ["Work", "Entrepreneurship", "Build in Public"]
excerpt_separator: <!--more-->
---

Eric Ries' book _The Lean Startup_ popularized the idea of the minimum viable product: the smallest version of a product that allows the creator to learn about the market, their customers' needs, and what the next steps should be. Over the last week, I've really shifted my idea of what "minimum" means with respect to [SleepEasy](https://www.sleepeasy.app)'s MVP—what's _really_ required for customers to get value out of it and for me to get feedback.

It started with asking myself: if I commit to shipping by December 1 to early access customers, what can I push back and what absolutely must be ready if the product is going to be worth paying for?

<!--more-->

Here are some things I've thrown overboard compared to my original grand (and not particularly time-constrained) vision:

1. A customer-facing web app
2. Self-service billing
3. Automated report emails
4. Automated alerting for customers

In short, I decided the core of the value SleepEasy provides has nothing to do with software. If customers get alerts when things break, that's the whole value proposition. Customers don't care that I manually triggered tests, or manually put together a report and emailed them, and so on. I'm taking the advice to "do things that don't scale" to heart.

That leaves the question: what _is_ required? The answer here comes down to anything I can't do manually (or that doesn't scale even to 10 early access customers):

- The ability to automatically scan web sites for broken links (it's not at all feasible for me to manually click thousands of links for each site)
- The ability to check the web site's homepage automatically to make sure the site isn't totally down (I don't have 24/7 availability to do this manually every 30 seconds 🙃)
- The ability to export data from the checks—especially the broken links check—into a format I can manually send to customers each week (in this case, I export a CSV and then upload it to Google Sheets to share with the customer)
- Automatic alerting for *me*—when there's an urgent incident like downtime, the app needs to page me (potentially at 3 am!) so I can either confirm it's a real issue (and manually alert the customer) or reject it and go back to sleep

Those are the things I really couldn't move forward without. In addition, there are a couple more "nice-to-haves" that don't really *need* to be automated at this point (they're things I could do manually), but I've granted myself an indulgence to include anyway because I bought the code to make them happen, and it's only a couple hours of work to integrate them. Those are:

- SSL/TLS certificate checking, so we can warn you before your certificate expires and HTTPS access breaks for everyone
- Domain name registration checking, so we can warn you before your domain expires and you lose access to it

For the fancier checks on the higher tiers—login, signup, billing, and visual checks—I'm planning to have no automation whatsoever to start with. I'm only anticipating two or three of these customers at first, so if I'm doing it manually, it's not going to be a huge burden.

I'm pretty happy with where I've ended up here, and I'm on track to have my "minimum" list finished even ahead of December 1. If that happens, I can focus on improving the reliability of the automated checks so that I'll receive fewer false-positive alerts. At this point, though, I'm trying not to look too much past that December 1 date. The next step after that might be a dashboard for customers to see the live status of their site, or a WordPress plugin to make it easy to get started. I'm confident, though, that contact with customers will provide me the direction I need.
