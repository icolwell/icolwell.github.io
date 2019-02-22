---
layout: post
title:  "Updating Multiple Namecheap Domains With ddclient - A History"
date:   2019-02-21 18:00:00 -0500
updated: 2019-02-22 14:00:00 -0500
categories: tech
tags: [server management, linux]
comments: true
---

This post is the result of my ddclient debug efforts.
I was trying figure out why I couln't update multiple namecheap domains even though existing info suggested it should work.
This post is a historical analysis of the problem, if you are looking for a solution to this issue, see my [other post](updating-multiple-namecheap-domains-with-ddclient).

## Introduction

If you are trying to update multiple namecheap domains and visit the [official namecheap documentation](https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient)
regarding how to configure ddclient, you will find moderators in the comments directing people towards [this blog post](https://thornelabs.blog/posts/linux-make-ddclient-work-with-multiple-namecheap-domains.html) from 2012.

The post describes how to patch ddclient 3.8.1 and older to accept multiple domain updates.
However, the post also describes that ddclient 3.8.3 does not need to be patched since the patch is included in 3.8.3.
Feeling confident that the issue was fixed years ago, I tried the latest version of ddclient (3.9.0) but I couldn't get it to work no matter what I tried.
After questioning both my ability to follow simple instructions and my own sanity, I decided to dig deeper and discovered the following.
I'll present it in a timeline format for easier reading.

## Timeline

**September 3rd, 2010**  
Robert Ian Hawdon creates a [blog post](https://robertianhawdon.me.uk/2010/09/03/making-ddclient-work-with-multiple-domains-on-namecheap/) that mentions the fact that ddclient is not able to update multiple different domains.
He also presents a ddclient patch to fix it.
His post seems to be based off of someone else's comment on the ddclient sourceforge forums (possibly [this post](https://sourceforge.net/p/ddclient/discussion/399428/thread/187e6520/#6bfd/05ae/dc99/eb0c) from 2007?).

**August 2012**  
ThorneLabs publishes a [blog post](https://thornelabs.blog/posts/linux-make-ddclient-work-with-multiple-namecheap-domains.html)
which just seems to be a more detailed copy of Robert's original post.
The proposed patch to ddclient is identical to Robert's (odd that he didn't credit Robert) except that it's aimed at version 3.8.1 (Robert's patch was for 3.8.0).

**Sometime after August 2012**  
People ask about multiple domain updates in the Namecheap official ddclient docs.
Users and Namecheap moderators refer them to the ThorneLabs blog post as a solution.

**October 8th, 2014**  
Robert submits a [github pull request](https://github.com/ddclient/ddclient/pull/7) to fix the issue once and for all.
The PR is accepted and ddclient 3.8.3 now updates multiple namecheap domains without the need to patch it, and there was much rejoicing.
However, Namecheap fails to update their documentation regarding all this and simply refers people to a now outdated patch method.

**May 4th, 2018**  
Then, for some reason that isn't clearly described in the [pull request](https://github.com/ddclient/ddclient/pull/54),
Robert's original fixes are [removed](https://github.com/ddclient/ddclient/pull/54/files)
so that ddclient's behaviour can be "aligned" with the
[official documentation](https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient).
The official documentation doesn't even mention anything about updating multiple domains (it only mentions multiple sub-domains), so I'm not sure what the intent of this PR was.
I guess if you consider the comment links to the old patch method to be part of the official documentation, then yes, I suppose removing good functionality so that it can be patched via the blog post again would be "aligning" ddclient with the offical docs.
I can't blame anyone for this except Namecheap, since they are the ones that haven't added these important details to their docs for 4+ years.
Unfortunately, this change made it into ddclient version 3.9.0, which means we are back to having to patch the latest version of ddclient or use an older version of it (3.8.3).

**August 29th, 2018**  
Eric Rucker submits a [bug report](https://sourceforge.net/p/ddclient/bugs/89/)
explaining the problem with [PR #54](https://github.com/ddclient/ddclient/pull/54) ([r204](https://sourceforge.net/p/ddclient/code/204/)).
I find this bug report and start to make sense of it all.
Thanks Eric!

## Summary

In summary, in seems that instead of simply updating the namecheap documentation regarding a new feature of ddclient (a new feature that the same docs showed you how to patch for!), ddclient was reverted back to a less functional state.
This way the old documentation could be followed which showed users how to manually patch in the feature themselves.

Surely no one really meant for it to turn out this way, but I can't help but laugh over how documentation ended up higher in priority than the software itself.

If you are looking for a solution to this issue, see [this post](updating-multiple-namecheap-domains-with-ddclient) that explains how to install a version of ddclient that is able to update multiple namecheap domains.
