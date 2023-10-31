---
title: "A Scary Story"
layout: post
authors: ['tyler']
categories: ["Work", "Build in Public"]
excerpt_separator: <!--more-->
image: "/assets/images/partial-eclipse.jpg"
---

In the spirit of Halloween, I'd like to tell you a scary story. This one, though, is all the more scary because it's _true_.

-----

It was a dark and stormy night, and two software developers were working late to ship a new internal service for their team. The finishing touch was to add production tokens for Google OAuth so their peers could sign in without manually creating an account.

They slowly opened the GCP console, the web page loading so slowly you could almost hear it _creeeeeeak_ across the wire.

"This storm must be messing with the wifi," said the senior dev. "Weird."

<!--more-->

The room was lit only with the soft, ethereal glow of their shared screen, and shadows danced ominously on the walls. The hum of their laptop's fan seemed louder than usual, reverberating with an eerie whine. The wind howled outside, rattling the windows as though the souls of deprecated APIs were yearning to break free.

The junior developer’s fingers hovered over the keyboard, trembling almost imperceptibly. "Alright," she muttered, "let's just get this auth token and get out here."

The console finally loaded, and the pair were greeted with a Lovecraftian maze of services, configuration, and secrets.

"Jamie said she'd get us access to the GCP dashboard for our internal services," said the senior dev. "I guess we're in."

They leaned in closer, studying the arcane terms inscribed in the sidebar.

"Credentials. This must be it."

Click.

"Why is there an existing token," wondered the junior aloud. "We don't have any other internal services that require login."

The senior dev gazed pensively out the window, into the rain pouring down in sheets, and thought for a time. "It must be from the old analytics dashboard that got retired last year. For security's sake, let's just cycle the token and use that one."

Click.

Click.

Click.

Lightning slashed the sky, illuminating the room with an otherworldly glow. The junior pasted the token into the configuration for the new service, saved, and shut down the computer. The vast emptiness of the office seemed to swallow sound as the pair headed for the exit. They parted ways at the door, and made their ways home, drenched and miserable.

It wasn't until the end of the following Monday that urgent whispers started to circulate through the office.

"Can you log in to prod?"

"That's weird... I wonder what's going on?"

"I just logged out, and I can't get in either."

"Is this broken for users too?"

Dear reader, it is with a heavy heart that I must tell you: the credentials they cycled were _not_ those of some long-dead, half-remembered internal service... but the very production credentials themselves! 😱

----------------------

Okay, melodramatic scary-stories-in-the-dark voice off.

The above really is based on a true story. Actually, it's based on _many_ such stories; some I've been told, and some I've unfortunately had a hand in.

When a miscommunication causes login or signup to be broken in production, the team often won't discover it for _way_ longer than anybody would like. There are a few contributing factors to that:

1. Team members don't tend to log in to the production site that often thanks to "remember me" cookies.
2. Customers aren't particularly likely to reach out when they have an issue like this—they'll either try again later or just give up and look for an alternative.
3. An issue with auth doesn't impact traditional uptime monitoring—after all, the site was still serving HTTP 200s!

This was a big part of my motivation for building a new take on website monitoring. One of the things we're going to offer is proactive testing of critical user flows, and we're doing it with humans in the loop. That fits with all three of our core tenets:

1. Dead simple (no need for you to write or maintain automated tests, which tend to break way more frequently than you'd like and require your time to fix)
2. More capable than other services (it's a helluva lot easier to point a bot at a page and go "does it still return a response?" than asking the more complex question of "am I able to go through the login flow start to finish, and does it work like a human would expect?")
3. No false alarms, ever—if a human is confused about whether login worked, you have either a real production incident or a UX disaster on your hands

If that sounds valuable for your company, drop your email below and I'll share updates as we get closer to inviting early access users!
