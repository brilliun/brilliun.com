---
title: "Building Microservices: Designing Fine-Grained Systems - Part III"
description: ""
keywords: []
tags: ['microservice', 'distributed system']
categories: ['Reading']
date: 2019-08-29T23:36:35+09:00

draft: false
isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

# Chapter 7. Testing

## Test Scope

### Trade-Offs

When you’re reading the pyramid, the key thing to take away is that as you go up the pyramid, the test scope increases, as does our confidence that the functionality being tested works. On the other hand, the feedback cycle time increases as the tests take longer to run, and when a test fails it can be harder to determine which functionality has broken. As you go down the pyramid, in general the tests become much faster, so we get much faster feedback cycles. We find broken functionality faster, our continuous integration builds are faster, and we are less likely to move on to a new task before finding out we have broken something. When those smaller-scoped tests fail, we also tend to know what broke, often exactly what line of code. On the flipside, we don’t get a lot of confidence that our system as a whole works if we’ve only tested one line of code! 

When broader-scoped tests like our service or end-to-end tests fail, we will try to write a fast unit test to catch that problem in the future. In that way, we are constantly trying to improve our feedback cycles.

### How Many?

A common anti-pattern is what is often referred to as a test snow cone, or inverted pyramid. Here, there are little to no small-scoped tests, with all the coverage in large-scoped tests. These projects often have glacially slow test runs, and very long feedback cycles. If these tests are run as part of continuous integration, you won’t get many builds, and the nature of the build times means that the build can stay broken for a long period when something does break.

## Consumer-Driven Tests to the Rescue 

What is one of the key problems we are trying to address when we use the integration tests outlined previously? We are trying to ensure that when we deploy a new service to production, our changes won’t break consumers. One way we can do this without requiring testing against the real consumer is by using a consumer-driven contract (CDC). With CDCs, we are defining the expectations of a consumer on a service (or producer).

## Testing After Production

### Mean Time to Repair Over Mean Time Between Failures? 

So by looking at techniques like blue/green deployment or canary releasing, we find a way to test closer to (or even in) production, and we also build tools to help us manage a failure if it occurs. Using these approaches is a tacit acknowledgment that we cannot spot and catch all problems before we actually release our software. 

Sometimes expending the same effort into getting better at remediation of a release can be significantly more beneficial than adding more automated functional tests. In the web operations world, this is often referred to as the trade-off between optimizing for mean time between failures (MTBF) and mean time to repair (MTTR).

# Chapter 8. Monitoring

## Service Metrics

There is an old adage that 80% of software features are never used. Now I can’t comment on how accurate that figure is, but as someone who has been developing software for nearly 20 years, I know that I have spent a lot of time on features that never actually get used.

# Chapter 11. Microservices at Scale

## Degrading Functionality

An essential part of building a resilient system, especially when your functionality is spread over a number of different microservices that may be up or down, is the ability to safely degrade functionality. Let’s imagine a standard web page on our ecommerce site. To pull together the various parts of that website, we might need several microservices to play a part. One microservice might display the details about the album being offered for sale. Another might show the price and stock level. And we’ll probably be showing shopping cart contents too, which may be yet another microservice. Now if one of those services is down, and that results in the whole web page being unavailable, then we have arguably made a system that is less resilient than one that requires only one service to be available.

What we need to do is understand the impact of each outage, and work out how to properly degrade functionality. If the shopping cart service is unavailable, we’re probably in a lot of trouble, but we could still show the web page with the listing. Perhaps we just hide the shopping cart or replace it with an icon saying “Be Back Soon!” 

With a single, monolithic application, we don’t have many decisions to make. System health is binary. But with a microservice architecture, we need to consider a much more nuanced situation. The right thing to do in any situation is often not a technical decision. We might know what is technically possible when the shopping cart is down, but unless we understand the business context we won’t understand what action we should be taking.

## Scaling Databases 

Scaling stateless microservices is fairly straightforward. Different types of databases provide different forms of scaling, and understanding what form suits your use case best will ensure you select the right database technology from the beginning.

### Availability of Service Versus Durability of Data 

Straight off, it is important to separate the concept of availability of the service from the durability of the data itself. You need to understand that these are two different things, and as such they will have different solutions. For example, I could store a copy of all data written to my database in a resilient filesystem. If the database goes down, my data isn’t lost, as I have a copy, but the database itself isn’t available, which may make my microservice unavailable too.

### Scaling for Reads 

Many services are read-mostly. Happily, scaling for reads is much easier than scaling for writes.

In a relational database management system (RDBMS) like MySQL or Postgres, data can be copied from a primary node to one or more replicas. This is often done to ensure that a copy of our data is kept safe, but we can also use it to distribute our reads. A service could direct all writes to the single primary node, but distribute reads to one or more read replicas.

The replication from the primary database to the replicas happens at some point after the write. This means that with this technique reads may sometimes see stale data until the replication has completed. Eventually the reads will see the consistent data. Such a setup is called eventually consistent, and if you can handle the temporary inconsistency it is a fairly easy and common way to help scale systems.

Years ago, using read replicas to scale was all the rage, although nowadays I would suggest you look to caching first, as it can deliver much more significant improvements in performance, often with less work.

### Scaling for Writes 

Reads are comparatively easy to scale. What about writes? One approach is to use sharding. With sharding, you have multiple database nodes. You take a piece of data to be written, apply some hashing function to the key of the data, and based on the result of the function learn where to send the data.

The complexity with sharding for writes comes from handling queries. Looking up an individual record is easy, as I can just apply the hashing function to find which instance the data should be on, and then retrieve it from the correct shard. But what about queries that span the data in multiple nodes — for example, finding all the customers who are over 18? If you want to query all shards, you either need to query each individual shard and join in memory, or have an alternative read store where both data sets are available. Often querying across shards is handled by an asynchronous mechanism, using cached results.

One of the questions that emerges with sharded systems is, what happens if I want to add an extra database node? In the past, this would often require significant downtime — especially for large clusters — as you might have to take the entire database down and rebalance the data. More recently, more systems support adding extra shards to a live system, where the rebalancing of data happens in the background.

As you may have inferred from this brief overview, scaling databases for writes are where things get very tricky, and where the capabilities of the various databases really start to become differentiated. I often see people changing database technology when they start hitting limits on how easily they can scale their existing write volume. If this happens to you, buying a bigger box is often the quickest way to solve the problem, but in the background you might want to look at systems like Cassandra, Mongo, or Riak to see if their alternative scaling models might offer you a better long-term solution.

## Caching

### Hiding the Origin 

With a normal cache, if a request results in a cache miss, the request goes on to the origin to fetch the fresh data with the caller blocking, waiting on the result. In the normal course of things, this is to be expected. But if we suffer a massive cache miss, perhaps because an entire machine (or group of machines) that provide our cache fail, a large number of requests will hit the origin. 

For those services that serve up highly cachable data, it is common for the origin itself to be scaled to handle only a fraction of the total traffic, as most requests get served out of memory by the caches that sit in front of the origin. If we suddenly get a thundering herd due to an entire cache region vanishing, our origin could be pummelled out of existence.

One way to protect the origin in such a situation is never to allow requests to go to the origin in the first place. Instead, the origin itself populates the cache asynchronously when needed, as shown in Figure 11-7. If a cache miss is caused, this triggers an event that the origin can pick up on, alerting it that it needs to repopulate the cache. So if an entire shard has vanished, we can rebuild the cache in the background. We could decide to block the original request waiting for the region to be repopulated, but this could cause contention on the cache itself, leading to further problems. It’s more likely if we are prioritizing keeping the system stable that we would fail the original request, but it would fail fast.

![Figure 11-7](/images/build-microservices/11-7.png)
*Figure 11-7. Hiding the origin from the client and populating the cache asynchronously

## CAP Theorem

It tells us that in a distributed system, we have three things we can trade off against each other: consistency, availability, and partition tolerance.

Consistency is the system characteristic by which I will get the same answer if I go to multiple nodes. Availability means that every request receives a response. Partition tolerance is the system’s ability to handle the fact that communication between its parts is sometimes impossible.

# Chapter 12. Bringing It All Together

## Principles of Microservices

### Model Around Business Concepts 

Experience has shown us that interfaces structured around business-bounded contexts are more stable than those structured around technical concepts. By modeling the domain in which our system operates, not only do we attempt to form more stable interfaces, but we also ensure that we are better able to reflect changes in business processes easily.

### Hide Internal Implementation Details 

To maximize the ability of one service to evolve independently of any others, it is vital that we hide implementation details. Modeling bounded contexts can help, as this helps us focus on those models that should be shared, and those that should be hidden. Services should also hide their databases to avoid falling into one of the most common sorts of coupling that can appear in traditional service-oriented architectures, and use data pumps or event data pumps to consolidate data across multiple services for reporting purposes.

## When Shouldn’t You Use Microservices? 

I get asked this question a lot. My first piece of advice would be that the less well you understand a domain, the harder it will be for you to find proper bounded contexts for your services. As we discussed previously, getting service boundaries wrong can result in having to make lots of changes in service-to-service collaboration — an expensive operation. So if you’re coming to a monolithic system for which you don’t understand the domain, spend some time learning what the system does first, and then look to identify clean module boundaries prior to splitting out services.
