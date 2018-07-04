---
layout: post
title: "Using Azure Table Storage and IQueryable"
date: 2015-11-23 10:00:00 -0700
tags: [azure, azure table storage, iqueryable, linq]
---

A recent task I was working on had me working with Microsoft's Azure Table Storage.

A recent realization was that we needed to have some filtering of the data that was being shown from table storage. I was the back end engineer assigned to review what could be done on the server side to provide a better way to filter. What we had was code that was not flexible enough for searching. It looked something like this:

{% highlight c# %}
public class AzureTableStorageRepository&lt;T&gt; : ITableStorageRepository&lt;T&gt; 
    where T : TableEntity, ITableStorageEntity, new()
{
    …

    public async Task&lt;IEnumerable&lt;T&gt;&gt; FindAsync(string partitionKey, 
        CancellationToken ct = default(CancellationToken), IList&lt;string&gt; projection = null)
    {
        var query = new TableQuery&lt;T&gt;()
            .Where(TableQuery.GenerateFilterCondition(
                "PartitionKey", QueryComparisons.Equal, partitionKey));

        if (projection != null)
        {
            query = query.Select(projection);
        }

        var items = await ExecuteSegmentedQuery(ct, query);

        return items;
    }

    async Task&lt;List&lt;T&gt;&gt; ExecuteSegmentedQuery(CancellationToken ct, TableQuery&lt;T&gt; query)
    {
        TableContinuationToken token = null;
        var items = new List&lt;T&gt;();
        do
        {
            var segment = await _cloudTable.ExecuteQuerySegmentedAsync(query, token, ct);
            token = segment.ContinuationToken;

            items.AddRange(segment.Results);
        } while (token != null &amp;&amp; !ct.IsCancellationRequested);
        return items;
    }

{% endhighlight %}

The code doesn't provide any searching on the server side. The consumer of the code is always forced to pass in a partition key and any real filtering has to be done in memory once the result is returned.

It's not much of a find method. The only real benefit here is that it will makes use of <code>ExecuteQuerySegmentedAsync</code> to page through results. That is a benefit because table storage can only return up to 1000 times at a time. Any more than that and an exception will be thrown. Chances are if you are using table storage, you are planning on having much more than 1000 rows per partition.

My first inclination was just to continue expanding upon using the <a href="https://msdn.microsoft.com/en-us/library/azure/dd894039.aspx">filter language that the Table Service API supports</a>. This means either the consumers of my code will have to pass up a string tailored to their requirements or I will have to support some sort of builder for these. And that sounded nasty. What other options were available?

I did some digging around and came across this announcement from September 2013: <a href="http://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/announcing-storage-client-library-2-1-rtm.aspx">Announcing Storage Client Library 2.1 RTM &amp; CTP for Windows Phone</a>. It is for Windows Phone specifically but it detailed what I was looking for. About half way down you will find "IQueryable Mode (2.1+)" and a table that shows Fluent Mode and IQueryable Mode. 

We were using the Fluent Mode in our solution to working with Table Storage and this shows exactly how to transform our queries. Perfect. Some quick changes and I'll be on my way. Now my FindAsync method looks like this:

{% highlight c# %}
public async Task&lt;IList&lt;TResult&gt;&gt; FindAsync&lt;TResult&gt;(Expression&lt;Func&lt;T, bool&gt;&gt; predicate, Expression&lt;Func&lt;T, TResult&gt;&gt; selector, CancellationToken cancellationToken = new CancellationToken())
{
    Ensure.ArgumentNotNull(predicate, "predicate");
    Ensure.ArgumentNotNull(selector, "selector");

    var query = _cloudTable.CreateQuery&lt;T&gt;().Where(predicate).Select(selector).AsTableQuery();

    var items = await ExecuteSegmentedQuery(cancellationToken, query);

    return items;
}
 
private async Task&lt;List&lt;TResult&gt;&gt;&gt; ExecuteSegmentedQuery&lt;TResult&gt;(CancellationToken ct, TableQuery&lt;TResult&gt; query)
{
    TableContinuationToken token = null;
    var items = new List&lt;TResult&gt;();
    do
    {
        var segment = await query.ExecuteSegmentedAsync(token, ct);
        token = segment.ContinuationToken;

        items.AddRange(segment.Results);
    } while (token != null &amp;&amp; !ct.IsCancellationRequested);
    return items;
}
{% endhighlight %}

I put together an integration test that passed with flying colors. Things were looking great.

Then I tried to pass a predicate that performed a <code>Contains</code> on a list of strings. And that's where the trouble started. <a href="https://msdn.microsoft.com/en-us/library/azure/dd135725.aspx">Query Operators Supported for the Table Service</a> shows what LINQ methods are available against the Table Service. To my disappointment, only 6 LINQ Query Operators are supported at this time. They are: <code>From</code>, <code>Where</code>, <code>Take</code>, <code>First</code>, <code>FirstOrDefault</code>, and <code>Select</code>. There are many that aren't supported but just a few to give you a hint: <code>Contains</code>, <code>Count</code>, <code>OrderBy</code>, <code>Single</code>, and <code>Distinct</code>. 

The Table Service supports very basic comparisons so this makes sense. But my LINQ focus thought process was disappointed. This can be overcome though. Let's just specify all times in a list individually for the predicate. Once I threw that together quickly, I was finished. 

Takeaway: This was a great exercise for me. It gave me some understanding of the Table Storage and I walked away with more experience than I had before. However, you really should be aware of some gotchas.

### If you use this incorrectly, there is a chance you are using Table Storage wrong.
Although, this is totally within the bounds of what is available to you, it's best to use a partition key and row key. That will always have the best performance in returning results. The next best query will just have a partition key. If the predicate you are passing is only filtering on the partition key and row key then you have nothing to worry about. If you are starting to filter on the dynamic data you are storing (read: any data other than partition key and row key), you might find the performance hindered more than you anticipated.

For our criteria, we are going to be querying against a single partition looking for data. It may slow down the response some but we anticipate that. However, if we are querying a single partition today, we open the doors to query the whole table tomorrow. That could bring on a slew of headaches.<br />
<blockquote class="tr_bq">
"In Windows Azure 
Tables, the string PartitionKey and RowKey properties work together as an index for your table. So when using Partition and Row Keys, the storage will use its index to find results really fast, while when using other entity properties the storage will result in table scan, significantly reducing performance."


<footer style="font-style: normal;">Ki Yi in <cite style="font-style: italic;" title="Improving Windows Azure Table Storage query performance"><a href="http://blogs.msdn.com/b/mast/archive/2013/02/19/improving-windows-azure-table-storage-query-performance.aspx">Improving Windows Azure Table Storage query performance</a></cite></footer></blockquote>
Use your best judgement when implementing your own solutions. A better understanding today will help you make good decisions for tomorrow.
