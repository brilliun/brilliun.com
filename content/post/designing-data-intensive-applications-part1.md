---
title: "Designing Data Intensive Applications Part1"
date: 2018-05-14T22:22:54+09:00
lastmod: 2018-05-14T22:22:54+09:00
draft: false
keywords: []
description: ""
tags: ['databases']
categories: ['Reading']

isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

<!-- toc -->

# CH01. Reliable, Scalable, and Maintainable Applications

## About Data System
We can think of any kind of databases, queues, or caches as a data system. Nowadays, these tools are optimized for a variety of different uses, and they no longer neatly fit into their origin categories

For example, there are data stores that are also used as message queues (Redis), and there are message queues with database-like durability guarantees (Apache Kafka). The boundaries between the categories are becoming blurred.

## Scalability, Load, and Performance

Load can be described with a few numbers which we call load parameters. The best choice of parameters depends on the architecture of your system: it may be requests per second to a web server, the ratio of reads to writes in a database, the number of simultaneously active users in a chat room, the hit rate on a cache, or something else.

## Twitter's Example

Two of Twitter's main operations are 1) *post tweet* and 2) *home timeline*, and there are two ways to implement them:

1. Posting a tweet simply inserts the new tweet into a global collection of tweets. When a user requests their home timeline, look up all the people they follow, find all the tweets for each of those users, and merge them (sorted by time). In a relational database, you could write a query such as: 
`SELECT tweets.*, users.* FROM tweets
 JOIN users ON tweets.sender_id = users.id
 JOIN follows ON follows.followee_id = users.id
 WHERE follows.follower_id = current_user;`

2. Maintain a cache for each user’s home timeline — like a mailbox of tweets for each recipient user. When a user posts a tweet, look up all the people who follow that user, and insert the new tweet into each of their home timeline caches. The request to read the home timeline is then cheap, because its result has been computed ahead of time. 

The first version of Twitter used approach 1, but the systems struggled to keep up with the load of home timeline queries, so the company switched to approach 2. This works better because the average rate of published tweets is almost two orders of magnitude lower than the rate of home timeline reads, and so in this case it’s preferable to do more work at write time and less at read time. 

However, the downside of approach 2 is that posting a tweet now requires a lot of extra work. On average, a tweet is delivered to about 75 followers, so 4.6k tweets per second become 345k writes per second to the home timeline caches. But this average hides the fact that the number of followers per user varies wildly, and some users have over 30 million followers. This means that a single tweet may result in over 30 million writes to home timelines! Doing this in a timely manner — Twitter tries to deliver tweets to followers within five seconds — is a significant challenge. In the example of Twitter, the distribution of followers per user (maybe weighted by how often those users tweet) is a key load parameter for discussing scalability, since it determines the fan-out load. Your application may have very different characteristics, but you can apply similar principles to reasoning about its load. 

The final twist of the Twitter anecdote: now that approach 2 is robustly implemented, Twitter is moving to a hybrid of both approaches. Most users’ tweets continue to be fanned out to home timelines at the time when they are posted, but a small number of users with a very large number of followers (i.e., celebrities) are excepted from this fan-out. Tweets from any celebrities that a user may follow are fetched separately and merged with that user’s home timeline when it is read, like in approach 1. This hybrid approach is able to deliver consistently good performance.

## Latency and Response Time

In a batch processing system such as Hadoop, we usually care about throughput — the number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size.iii In online systems, what’s usually more important is the service’s response time — that is, the time between a client sending a request and receiving a response. 

Latency and response time are often used synonymously, but they are not the same. The response time is what the client sees: besides the actual time to process the request (the service time), it includes network delays and queueing delays. Latency is the duration that a request is waiting to be handled — during which it is latent, awaiting service.

## Approaches for Scaling Up

While distributing stateless services across multiple machines is fairly straightforward, taking stateful data systems from a single node to a distributed setup can introduce a lot of additional complexity. For this reason, common wisdom until recently was to keep your database on a single node (scale up) until scaling cost or high- availability requirements forced you to make it distributed. 

# CH02. Data Models and Query Languages

In a complex application there may be many intermediary levels, such as underlying databases, application-specific models, APIs, and even APIs built upon APIs, but the basic idea is the same: each layer hides the complexity of the layers below it by providing a clean data model.

This chapter is talking about a range of general-purpose data models for **data storage** and **querying**. They are the **relational model**, the **document model**, and the **graph-based data model**.

## Relational Model vs. Document Model

### Dominance of Relational Model
In early times, other databases forced application developers to think a lot about the internal representation of the data in the database, while the goal of the relational model was to hide that implementation detail behind a cleaner interface. So relational model turns out to generalize very well, beyond their original scope of business data processing, to a broad variety of use cases, making it the most widely used database until today.

### The Birth of NoSQL
There are several driving forces behind the adoption of NoSQL databases, including: 

* A need for greater scalability than relational databases can easily achieve, including very large datasets or very high write throughput * A widespread preference for free and open source software over commercial database products
* Specialized query operations that are not well supported by the relational model
* Frustration with the restrictiveness of relational schemas, and a desire for a more dynamic and expressive data model

### The Object-Relational Mismatch

Most application development today is done in object-oriented programming languages, which leads to a common criticism of the SQL data model: if data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns. The disconnect between the models is sometimes called an impedance mismatch.

Object-relational mapping (ORM) frameworks like ActiveRecord and Hibernate reduce the amount of boilerplate code required for this translation layer, but they can’t completely hide the differences between the two models.

For a data structure like a resume, which is mostly a self-contained document, a JSON representation can be quite appropriate. It has better locality than the multi-table schema. If you want to fetch a resume in the relational example, you need to either perform multiple queries (query each table by user_id) or perform a messy multi-way join between the users table and its subordinate tables. In the JSON representation, all the relevant information is in one place, and one query is sufficient.

### Many-to-one and Many-to-many Relationships
Database normalization requires many-to-one relationships, which don't fit nicely into the document model. In relational databases, it is normal to refer to rows in other tables by ID, because joins are easy. In document databases, support for joins is often weak, sometimes you even need to emulate a join in application code by making multiple queries.

It’s possible to reduce the need for joins by denormalizing, but then the application code needs to do additional work to keep the denormalized data consistent. Joins can be emulated in application code by making multiple requests to the database, but that also moves complexity into the application and is usually slower than a join performed by specialized code inside the database. In such cases, using a document model can lead to significantly more complex application code and worse performance

### Schemaless

Document databases are sometimes called *schemaless*, while in relational databases, schema changes have a bad reputation of being slow and requiring downtime. This reputation is not entirely deserved: most relational database systems execute the `ALTER TABLE` statement in a few milliseconds. MySQL is a notable exception — it copies the entire table on `ALTER TABLE`, which can mean minutes or even hours of downtime when altering a large table.

### Data Locality for Queries

A document is usually stored as a single continuous string, encoded as JSON, XML, or a binary variant thereof (such as MongoDB’s BSON). If your application often needs to access the entire document (for example, to render it on a web page), there is a performance advantage to this storage locality. If data is split across multiple tables, multiple index lookups are required to retrieve it all, which may require more disk seeks and take more time. 

The locality advantage only applies if you need large parts of the document at the same time. The database typically needs to load the entire document, even if you access only a small portion of it, which can be wasteful on large documents. On updates to a document, the entire document usually needs to be rewritten — only modifications that don’t change the encoded size of a document can easily be performed in place. For these reasons, it is generally recommended that you keep documents fairly small and avoid writes that increase the size of a document. These performance limitations significantly reduce the set of situations in which document databases are useful.

It’s worth pointing out that the idea of grouping related data together for locality is not limited to the document model. For example, Google’s Spanner database offers the same locality properties in a relational data model, by allowing the schema to declare that a table’s rows should be interleaved (nested) within a parent table.

## Conclusion

The main arguments in favor of the document data model are schema flexibility, better performance due to locality, and that for some applications it is closer to the data structures used by the application. The relational model counters by providing better support for joins, and many-to-one and many-to-many relationships.

# CH03. Storage and Retrieval

Why should application developers care how the database handles storage and retrieval internally? Because there is a big difference between storage engines that are optimized for transactional workloads and those that are optimized for analytics.

There are two families of commonly used storage engines: *log-structured storage engines*, and *page-oriented storage engines* such as B-trees.

