---
title: Migrating a Large SVN Repository to a New Server
categories: ['Version Control']
authors: ['tyler']
layout: post
---

We ran into a situation at work where we needed to move our SVN repo from an old Linux server (running Ubuntu 10.04, in 2017!!) to a shiny new cloud instance.

[Jump to the TL;DR](#instructions)

Dear God, don’t do this
-----------------------

The standard advice you see on the web is to use `svn dump` on the old server, transfer the dump file, then use `svn load` on the new server. That works fine for small repos, where the total transfer time will be negligible no matter how you do it, but for a large repo, it’s a disaster. Time estimates I’ve seen for this around the web say it’ll take roughly 1 hour per GB to dump, then another hour per GB to load… not to mention the fact that the dump files are _larger_ overall than the raw filesystem data, so the transfer itself is slower!

Using rsync
-----------

Not wanting to spend ~150 hours on this for our 70 GB repo, I wanted to try moving _just_ the raw filesystem data. [This StackOverflow answer](https://stackoverflow.com/a/7669096/1417451) indicated such a thing could work using `scp`, but of course there are two scary things that could happen:

1.  What happens if you have a network connection hiccup mid-transfer? (Nobody wants to start over!)
2.  What happens if somebody adds a new commit while you’re working? (You could, at an organizational level, ask for a “lock” for the hours you need to do the transfer, but it’d be nice not to impede everyone’s work for that long.)

The solution, of course, is to use `rsync`! You run it once to transfer your directory initially, _then_ obtain a lock; then you run it a second time to pick up any changes you missed from the first (long) transfer. Then it’s just a matter of upgrading the repo and preparing it for use.

<p id="instructions">So, from beginning to end, the complete steps are:</p>

1.  On your old server:  
    `$ rsync -aPz /path/to/svn-repo/ username@newServer:/destination/path/`
2.  Email your team to get a “lock” (no more committing to the old server!)
3.  Once more:  
    `$ rsync -aPz /path/to/svn-repo/ username@newServer:/destination/path/   `(picking up any changes that were made during the first copy)
4.  On the new server:  
    `$ svnadmin upgrade /destination/path/`  
    `$ svnadmin verify /destination/path/`
