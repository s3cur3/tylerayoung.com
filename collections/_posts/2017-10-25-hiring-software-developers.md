---
title: Hiring Software Developers
layout: post
authors: ['tyler']
categories: ["Work"]
summary: "The traditional hiring process—source a billion resumes, eliminate them based on keywords, interview the top hundred people, then pick one—is pretty broken. Fundamentally, filtering by pre-existing experience with your tech makes you miss the enormous swath of smart people who _could_ get up to speed on your stack quickly, and potentially become tremendously valuable to you. Whenever I hear people lament that it’s impossible to hire developers, my first question is always: how many good devs are you excluding from your search?"
---

The traditional hiring process—source a billion resumes, eliminate them based on keywords, interview the top hundred people, then pick one—is pretty broken. Fundamentally, filtering by pre-existing experience with your tech makes you miss the enormous swath of smart people who _could_ get up to speed on your stack quickly, and potentially become tremendously valuable to you. Whenever I hear people lament that it’s impossible to hire developers, my first question is always: how many good devs are you excluding from your search?

In addition to unduly weighing resume keywords, the traditional interview process also massively penalizes people who can’t code or problem solve under pressure. There’s a certain class of people who do well in high-pressure interviews, but that skill isn’t strongly correlated with overall value as a developer.

The _alternative_ to all this is: **just have people write code that exemplifies the role you’re hiring for**. We wanted anyone who _might_ be a good fit to apply, and we would narrow our choices by comparing the code they wrote.

At a high level, our interview process looked like this:

1.  [A blog post asking for applicants](http://developer.x-plane.com/2017/02/were-hiring/). (This obviously only worked because we have a large number of developers reading our blog; I think the same principles could have applied to a StackOverflow job listing and the like.)
2.  A preliminary chat over email, to answer any questions they have and generally make sure they aren’t vastly under-qualified.
3.  A small set of “homework” problems—problems that are characteristic of the most difficult type of work they’d be doing, with a definite right answer (or set of right answers).
4.  A phone interview with the top performers on the “homework” questions.

Let me break down how we handled each of these stages…

The call for applications
-------------------------

In our blog post, we were very careful not to eliminate any potential hires. We wanted to hear from _any_ smart, self-disciplined developer, period, and let the homework problems show us who to pursue.

In retrospect, specify a CS degree as a requirement was a mistake, one we won’t make next time; one of the two candidates we hired doesn’t have a degree, and he’s one of the strongest developers I’ve ever met. I hate to think that we would have weeded him out for no reason at all if he hadn’t had the self-confidence to apply anyway. Strong self-confidence is not a job requirement!

The instructions in the post were simple: send us an email introduction  with a brief overview of a project (or projects) you’ve enjoyed working on, and a discussion of projects you have _not_ enjoyed working on. I tried to make it clear: there’s no need to stress about this email!

One final note about the call for applicants: this is also a time to _sell your company_.

The intro email
---------------

In the intro email the applicants sent, I was looking for a few things:

*   Past projects (not necessarily professional) that indicated this person knew software development. I didn’t follow up with the people whose projects amounted to “hello world.”
*   Interest in the type of work they’d be doing. If the person’s description of what they were interested in directly conflicted with the job we were trying to hire for, I sent back a response gently explaining why I didn’t think this would be a good fit.
*   Strong English communication skills. We’re an English speaking team, and especially for a development role, we can’t afford to have ongoing communication issues.
*   Ability to follow simple directions. This one was inadvertent, but I got quite a few emails lacking the details we requested in the blog post. I did not follow up with these people.

I also used the email chat to answer any questions they have and generally make sure they aren’t vastly under-qualified. My primary goal at this stage was to not eliminate anyone who could potentially knock our socks off in the coding challenge… but at the same time, to not waste anyone’s time.

I was also up front at this stage with people I expected we couldn’t offer competitive compensatation. For instance, a developer with 15 years experience in database programming working in Silicon Valley probably commands $150k+, but a) as a distributed company, we’re not going to pay SV premiums, and b) their (vast) experience is in a _totally_ unrelated field. Some applicants were okay moving forward with the interview processs despite the expected pay cut, some were not.

The homework problems
---------------------

The most important feature of the homework problems is that they had _objective_ criteria on which they could be compared. Each problem had _dozens_ of scoring criteria, such that anyone on our dev team could grade them and we could differentiate a really good solution from an okay one. And because we gave the same problems to everyone, we could compare their performance to the rest of the applicant pool.

I was aiming for a total time cost to the interviewees of under 2 hours for people with prior experience in the problem domain (in this case, performance tuning an algorithm in C++). But, for people who had never worked on something like this, they might have needed a significant amount of reading to get up to speed—whether that’s a primer on what sorts of things govern an algorithm’s performance on modern CPUs, or simply a crash course in C++. I did my best to provide any resources the applicants needed—again, I wasn’t trying to filter by previous experience!—such that a smart applicant could read the provided reference material and turn it into a strong homework submission given enough time. (For applicants that didn’t want to invest the time to go from zero knowledge to a working solution, I completely understood… but obviously we couldn’t move forward with those people.)

The interview
-------------

We scheduled Skype interviews with the six candidates who submitted the best homework solutions. Each interview was conducted by two to three senior devs on the team, in a (hopefully) low-stress discussion format.

Since the homework told us everything we needed to know about the person’s ability to code, the interview was focused on their ability to communicate about technical issues. We talked about their homework solutions, asked about their past work, etc. We also did our best to make sure they would be a good fit for the level of independence and autonomy that we need from our developers.

A few of my favorite questions we asked in person:

*   Can you tell us more about some of the challenges you faced on \[some past project\]? (This question works best if you can ask about a specific subsystem that you, the interviewer, have some domain knowledge on.)
*   In any sufficiently large codebase, you tend to accumulate some old, awful, legacy code that’s kept around because “it ain’t broke.” As a developer, it feels skeezy to leave that code in such a horrendous state, but you probably don’t need to touch it in order to implement any given feature/fix. What do you do? (My goal with this question was to assess how they balanced _business_ needs against personal “hygeine.”)
*   Development teams operate on a continuum between “total developer anarchy” and “micromanagement of every task.” Where do you prefer to be along that continuum? (This question is trying to ensure the person won’t expect more oversight than we can give, and I tend to follow up with questions about how they’ve handled management in the past.)

Results
-------

We were _extremely_ happy with how this worked out. The two people we ended up hiring are phenomenal, but honestly, I expect that any of the people that made it to the phone interview would have been great hires… and most of those would have been filtered out in step 1 if we had posted the typical list of “job requirements.”
