---
layout: post
title: "Microsoft Azure Best Practices"
date: 2016-07-21 09:00:00 -0700
tags: [c#, ide, visual studio, jetbrains, resharper]
---

Microsoft Azure best practices can be found all over. I want to make sure that I hit all the same great points that have been made in other blogs in place that I can find quickly (out of a personal desire). Certainly another page describing the practices never hinders engineers from stumbling upon these.

When interacting with the blob and table storage, there are several considerations to take in.

1. The default connection limit is incredibly bad. It has a default value of two (2). This means only 2 sockets can be open at any given domain.
2. The Nagle algorithm is great for messages that are greater that 14 kilobytes but are detrimental for messages smaller than this limit.
3. Another recommendation is to disable the expect 100 Continue response for requests you expect to succeed.
4. Increase ThreadPool minimum number of threads if using synchronous code with Async Tasks

Include this set of code when initializing your connection to Azure (each line corresponds to the numbered item above):

{% highlight c# %}

ServicePointManager.UseNagleAlgorithm = false;
ServicePointManager.Expect100Continue = false;
ServicePointManager.DefaultConnectionLimit = 100;
ThreadPool.SetMinThreads(100,100); //(Determine the right number for your application)

{% endhighlight %}

These are a good bare minimum for configuring your application to communicate with the Microsoft Azure platform. Microsoft keeps a great document that details this information and much more. If you are working with the Azure platform, it would be hugely beneficial to you to check it out. Since Microsoft now has that documentation, it has become a key document to review anytime I setup a new project on Azure. Since it is maintained by Microsoft, it will always be up to date with the latest information. No more wondering about blog posts from years ago.