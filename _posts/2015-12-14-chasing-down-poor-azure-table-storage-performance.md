---
layout: post
title: "Chasing down poor Azure Table Storage performance"
date: 2015-12-14 17:00:00 -0700
tags: [azure, azure table storage, iqueryable, linq]
---

A coworker and I have been puzzled for the last week or so. We have been seeing incredibly slow table storage performance.(···) Here is the problem that we faced:

{% highlight c# %}

public class OurEntity : TableEntity
{
    public string PrimaryId
    {
        get { return PartitionKey; }
        set { PartitionKey = value; }
    }
    
    public string SecondaryId
    {
        get { return RowKey; }
        set { RowKey = value; }
    }
    ...
}

{% endhighlight %}

We created a class that inherited from TableEntity and had more qualified names for our Partition and Row Keys. We weren't aware of any implications when taking this approach.

We were then going on to use these properties in our IQueryable searches:

    _entityRepository.FindAsync(x => x.PrimaryId == "ThatOneId" && x.SecondaryId == "ThatOtherId");

We were certainly surprised at the results. We loaded data into a test environment and fired off our integration tests. You can imagine the look on my face when I saw that one query took 20 minutes to complete. My coworker realized when looking through table storage that our properties which were just supposed to be PartitionKey and RowKey were actually stored in table storage under their own slot.

So if you find yourself creating new properties to give PartitionKey and RowKey more qualifying names, make sure you use this:

{% highlight c# %}

public class OurEntity : TableEntity
{
    [IgnoreProperty]
    public string PrimaryId
    {
        get { return PartitionKey; }
        set { PartitionKey = value; }
    }
    
    [IgnoreProperty]
    public string SecondaryId
    {
        get { return RowKey; }
        set { RowKey = value; }
    }
    ...
}

{% endhighlight %}

    _entityRepository.FindAsync(x => x.PartitionKey == "ThatOneId" && x.RowKey == "ThatOtherId");


Trust me, it will save you plenty of headaches.

### Update
I didn't get a chance to dive down into what was happening when I originally put this post up. My coworker came across this problem while reviewing the data that was in table storage while using Visual Studio. If you head to your Server Explorer and connect to your Azure Table Storage, you can view the rows that are currently available within table storage. While reviewing this data, he noticed two columns in the data called "PrimaryId" and "SecondaryId." At first glance, it didn't seem to be much of an issue. However, as we began chasing down this slowness, it dawned on him that this may be the root of our issues.  

Having the data duplicated in table storage isn't a problem. However, when you are making queries believing you are querying the PartitionKey, that is when this problem emerges. Our queries were against these PrimaryId properties. That means it wasn't using the PartitionKey to query the storage. That's right! Full table searches all around!  

I don't anticipate many individuals running into this problem but it was enough of a problem for us that I had to document it.
