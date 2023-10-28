---
title: "Going Indie and Building in Public"
layout: post
authors: ['tyler']
categories: ["Work", "Build in Public", "Entrepreneurship"]
image: "/assets/images/san-diego-2018/2018-12-23-14.17.38-1_hdr-sailboat.jpg"
excerpt_separator: <!--more-->
---

I sort of quit my job.

I say "sort of" because I'm going to be gradually winding down my work on <a href="https://felt.com" rel="nofollow">Felt</a>, where I've been for the past couple years, in order to gradually ramp _up_ the amount of time I spend working on my own business.

That business is a new take on website monitoring. Existing monitoring focuses on uptime ("is my site down"), which is fine, and sort of table stakes. But I think there's a market for being way, _way_ more proactive about monitoring... alerting you before there are problems, or alerting you about things that might be damaging to your business or your reputation, but which aren't captured by the most basic "is the site responding at all?" checks. At the same time, existing solutions are just _massively_ more complex than they need to be... starting from the way they bill ("$_x_/month for _y_ monitors") through the amount of configuration required to get them to _just do what you want_ ("tell me if my site is broken"). And don't even get me started on the false alarms... in the course of my career, I've had maybe 10&times; more wakeup calls at 2 in the morning from false alarms in a monitoring service than actual issues with my website!

So I'm building a site monitoring service... it's going to be:

* Absolutely dead simple—just tell us the web site you want us to monitor and we do the rest
* _Way_ more capable than any other service on the market (for instance, maybe your web site is online, but one of your business's key user flows isn't working—login, new account signup, checkout, billing, etc.—we'll alert you about that)
* Guaranteed to never produce a false alarm—if you get a 2 am phone call, it's because your stuff is _actually_ having a critical incident!

<!--more-->

If that sounds like something you're interested in, you can sign up below to get a notification when it's ready.

## Why this idea?

Every software developer fantasizes about one day starting their own SaaS, and this was the first idea I've had that both

* addresses a problem I know from my own experience is a burning need at every place I've worked at, _and_
* is in an area I feel uniquely qualified to address (I've been banging the automated testing drum my whole career, and I actually built a very limited, crappy version of this at a past company).

I can't say for sure it'll work out, but I think it's the best shot I've ever had.

## Why now?

The standard advice is that if you want to bootstrap a business, you should do it on the side until it can replace or nearly replace your normal full-time income. That's totally reasonable advice in my book.

Personally, though, after spending a month doing nights and weekends on it, I realized I wasn't making the kind of progress on it that I wanted. Being a father of two young kids, "nights and weekends" doesn't actually amount to very much free time! It felt like... not that it was "now or never," but rather that now was as good a time as any, and if I didn't make the jump, this business just wasn't going to happen.

So I'm doing something I've done rather a lot in my life... making a major life decision I'd advise _most_ people against.

Between my existing savings and contracting work I have lined up, I'll have enough in savings to cover my family's rather low "burn rate" for about 10 months. The plan is to pick up contracting work a couple days a week after that to make ends meet. I'm extremely fortunate as a mid-career tech nerd to make enough per hour to be able to get by on a couple days a week, and to have been able to save enough to make this not a massive risk for my family.

## Wanna follow along?

I'm not quite ready to link to a landing page for the monitoring service... it needs some TLC before it's in a state that's only mildly embarrassing rather than totally mortifying. But I'm going to be building this thing in public, talking about the journey on this blog, [Twitter](https://twitter.com/TylerAYoung) (I refuse to call it anything else, dammit), and [Mastodon](https://fosstodon.org/@tylerayoung). I'll talk about:

- How I think about the product positioning and marketing
- The ups and downs of doing sales
- Interesting bits about how the service is built
- Actual revenue numbers?? (All the cool kids are doing it, even though it freaks me out a bit...)

If you'd like an email when a new post goes out—including one when the service is open for business!—drop your email here!

<form
  method="POST"
  action="https://customerioforms.com/forms/submit_action?site_id=c4a416941ebb499f636d&form_id=66ffc0e21d3d4cb&success_url=https://tylerayoung.com/signup-successful/"
  class="max-w-xl bg-green-100 p-2 sm:p-8 m-2 sm:m-4 rounded"
>
    <h2 style="margin-top: 0; margin-bottom: 0.25em;">Follow the #BuildInPublic journey</h2>
    <p>1&ndash;2 emails a week max, spam-free. Unsubscribe at any time.</p>
    <div class="flex flex-row gap-x-2 items-center">
        <label for="email_input" class="block font-medium leading-6 text-gray-900 w-fit">Email</label>
        <div class="relative rounded-md shadow-sm" style="flex-grow: 1">
            <div class="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3">
                <svg class="h-5 w-5 text-gray-400" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                    <path d="M3 4a2 2 0 00-2 2v1.161l8.441 4.221a1.25 1.25 0 001.118 0L19 7.162V6a2 2 0 00-2-2H3z" />
                    <path d="M19 8.839l-7.77 3.885a2.75 2.75 0 01-2.46 0L1 8.839V14a2 2 0 002 2h14a2 2 0 002-2V8.839z" />
                </svg>
            </div>
            <input type="email" name="email" id="email_input" class="block grow w-full rounded-md border-0 py-1.5 pl-10 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-green-600 sm:text-sm sm:leading-6" placeholder="you@example.com">
        </div>
    </div>
    <button type="submit" class="mt-4 flex w-full justify-center rounded-md bg-green-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white shadow-sm hover:bg-green-700 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-green-600">Send me new #BuildInPublic posts!</button>
</form>
