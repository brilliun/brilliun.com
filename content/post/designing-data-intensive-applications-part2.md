
---
title: "Designing Data Intensive Applications - Part II"
description: ""
keywords: []
tags: ['database', 'distributed system']
categories: ['Reading']
date: 2019-12-09T21:48:35+09:00
lastmod: 2019-12-10T00:49:54+09:00

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

<!-- toc -->


# CH03. Storage and Retrieval

Why should application developers care how the database handles storage and retrieval internally? Because there is a big difference between storage engines that are optimized for transactional workloads and those that are optimized for analytics.

There are two families of commonly used storage engines: *log-structured storage engines*, and *page-oriented storage engines* such as B-trees.

## Data Structures That Power Your Database 

Consider the world’s simplest database, implemented as two Bash functions:

```
#!/bin/bash 
db_set () {
    echo "$1,$2" >> database
}

db_get () {
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

These two functions implement a key-value store. You can call `db_set key value`, which will store `key` and `value` in the database.

Our db_set function actually has pretty good performance for something that is so simple, because appending to a file is generally very efficient. Similarly to what db_set does, many databases internally use a log, which is an append-only data file. Real databases have more issues to deal with (such as concurrency control, reclaiming disk space so that the log doesn’t grow forever, and handling errors and partially written records), but the basic principle is the same.

On the other hand, our db_get function has terrible performance if you have a large number of records in your database. Every time you want to look up a key, db_get has to scan the entire database file from beginning to end, looking for occurrences of the key. In algorithmic terms, the cost of a lookup is O(n): if you double the number of records n in your database, a lookup takes twice as long. That’s not good.

In order to efficiently find the value for a particular key in the database, we need a different data structure: an index. The general idea behind them is to keep some additional metadata on the side, which acts as a signpost and helps you to locate the data you want. If you want to search the same data in several different ways, you may need several different indexes on different parts of the data.

### Hash Indexes 

Let’s start with indexes for key-value data. Key-value stores are quite similar to the dictionary type that you can find in most programming languages, and which is usually implemented as a hash map (hash table).

Let’s say our data storage consists only of appending to a file, as in the preceding example. Then the simplest possible indexing strategy is this: keep an in-memory hash map where every key is mapped to a byte offset in the data file — the location at which the value can be found, as illustrated in Figure 3-1.

![Figure 3-1](/images/designing-data-intensive-applications/3-1.png)

As described so far, we only ever append to a file — so how do we avoid eventually running out of disk space? A good solution is to break the log into segments of a certain size by closing a segment file when it reaches a certain size, and making subsequent writes to a new segment file. We can then perform compaction on these segments, as illustrated in Figure 3-2. Compaction means throwing away duplicate keys in the log, and keeping only the most recent update for each key.

![Figure 3-2](/images/designing-data-intensive-applications/3-2.png)

Moreover, since compaction often makes segments much smaller (assuming that a key is overwritten several times on average within one segment), we can also merge several segments together at the same time as performing the compaction, as shown in Figure 3-3. Segments are never modified after they have been written, so the merged segment is written to a new file. The merging and compaction of segments can be done in a background thread, and while it is going on, we can still continue to serve read requests using the old segment files, and write requests to the latest segment file. After the merging process is complete, we switch read requests to using the new merged segment instead of the old segments — and then the old segment files can simply be deleted.

![Figure 3-3](/images/designing-data-intensive-applications/3-3.png)

Each segment now has its own in-memory hash table, mapping keys to file offsets. In order to find the value for a key, we first check the most recent segment’s hash map; if the key is not present we check the second-most-recent segment, and so on. The merging process keeps the number of segments small, so lookups don’t need to check many hash maps.

An append-only log seems wasteful at first glance: why don’t you update the file in place, overwriting the old value with the new value? But an append-only design turns out to be good for several reasons: 

* Appending and segment merging are sequential write operations, which are generally much faster than random writes.
* Concurrency and crash recovery are much simpler if segment files are append-only or immutable.
* Merging old segments avoids the problem of data files getting fragmented over time.

However, the hash table index also has limitations: 

* The hash table must fit in memory, so if you have a very large number of keys, you’re out of luck.
* Range queries are not efficient.

### SSTables and LSM-Trees 

In Figure 3-3, each log-structured storage segment is a sequence of key-value pairs. These pairs appear in the order that they were written, the order of key-value pairs in the file does not matter.

Now we can make a simple change to the format of our segment files: we require that the sequence of key-value pairs is sorted by key. We call this format Sorted String Table, or SSTable for short. With this format, we cannot append new key-value pairs to the segment immediately, since writes can occur in any order.

SSTables have several big advantages over log segments with hash indexes: 

1. Merging segments is simple and efficient, even if the files are bigger than the available memory. The approach is like the one used in the mergesort algorithm and is illustrated in Figure 3-4.

![Figure 3-4](/images/designing-data-intensive-applications/3-4.png)

2. In order to find a particular key in the file, you no longer need to keep an index of all the keys in memory. See Figure 3-5 for an example: say you’re looking for the key handiwork, you can jump to the offset for handbag and scan from there until you find handiwork.

#### Constructing and maintaining SSTables 

Fine so far — but how do you get your data to be sorted by key in the first place? Maintaining a sorted structure on disk is possible (see “B-Trees”), but maintaining it in memory is much easier. There are plenty of well-known tree data structures that you can use, such as red-black trees or AVL trees [2]. With these data structures, you can insert keys in any order and read them back in sorted order.

We can now make our storage engine work as follows: 

* When a write comes in, add it to an in-memory balanced tree data structure (for example, a red-black tree). This in-memory tree is sometimes called a memtable. 
* When the memtable gets bigger than some threshold — typically a few megabytes — write it out to disk as an SSTable file. This can be done efficiently because the tree already maintains the key-value pairs sorted by key. The new SSTable file becomes the most recent segment of the database. While the SSTable is being written out to disk, writes can continue to a new memtable instance. 
* In order to serve a read request, first try to find the key in the memtable, then in the most recent on-disk segment, then in the next-older segment, etc. 
* From time to time, run a merging and compaction process in the background to combine segment files and to discard overwritten or deleted values. 

This scheme works very well. It only suffers from one problem: if the database crashes, the most recent writes (which are in the memtable but not yet written out to disk) are lost. In order to avoid that problem, we can keep a separate log on disk to which every write is immediately appended, just like in the previous section. That log is not in sorted order, but that doesn’t matter, because its only purpose is to restore the memtable after a crash.

#### Making an LSM-tree out of SSTables

Originally this indexing structure was described by Patrick O’Neil et al. under the name Log-Structured Merge-Tree (or LSM-Tree). Storage engines that are based on this principle of merging and compacting sorted files are often called LSM storage engines.

#### Performance optimizations

Even though there are many subtleties, the basic idea of LSM-trees — keeping a cascade of SSTables that are merged in the background — is simple and effective.

### B-Trees

Like SSTables, B-trees keep key-value pairs sorted by key, which allows efficient key-value lookups and range queries.

One page is designated as the root of the B-tree; whenever you want to look up a key in the index, you start here. The page contains several keys and references to child pages.

Eventually we get down to a page containing individual keys (a leaf page), which either contains the value for each key inline or contains references to the pages where the values can be found.

#### Making B-trees reliable

In order to make the database resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a write-ahead log (WAL, also known as a redo log). This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself. When the database comes back up after a crash, this log is used to restore the B-tree back to a consistent state.

### Comparing B-trees and LSM-Trees

Even though B-tree implementations are generally more mature than LSM-tree implementations, LSM-trees are also interesting due to their performance characteristics. As a rule of thumb, LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads. Reads are typically slower on LSM-trees because they have to check several different data structures and SSTables at different stages for compaction.

#### Downside of LSM-trees

A downside of log-structureed storage is that the compaction process can sometimes interfere with performance of ongoing reads and writes.

### Other Index Structures

#### Storing values within the index 

The key in an index is the thing that queries search for, but the value can be one of two things: it could be the actual row (document, vertex) in question, or it could be a reference to the row stored elsewhere. In the latter case, the place where rows are stored is known as a heap file, and it stores data in no particular order (it may be append-only, or it may keep track of deleted rows in order to overwrite them with new data later). The heap file approach is common because it avoids duplicating data when multiple secondary indexes are present: each index just references a location in the heap file, and the actual data is kept in one place.

In some situations, the extra hop from the index to the heap file is too much of a performance penalty for reads, so it can be desirable to store the indexed row directly within an index. This is known as a clustered index. For example, in MySQL’s InnoDB storage engine, the primary key of a table is always a clustered index, and secondary indexes refer to the primary key (rather than a heap file location).

#### Keeping everything in memory

Disks have two significant advantages: they are durable (their contents are not lost if the power is turned off), and they have a lower cost per gigabyte than RAM.

As RAM becomes cheaper, the cost-per-gigabyte argument is eroded. Many datasets are simply not that big, so it’s quite feasible to keep them entirely in memory, potentially distributed across several machines. This has led to the development of in-memory databases.

Some in-memory key-value stores, such as Memcached, are intended for caching use only, where it’s acceptable for data to be lost if a machine is restarted. But other in-memory databases aim for durability, which can be achieved with special hardware (such as battery-powered RAM), by writing a log of changes to disk, by writing periodic snapshots to disk, or by replicating the in-memory state to other machines.

The performance advantage of in-memory databases is not due to the fact that they don’t need to read from disk. Even a disk-based storage engine may never need to read from disk if you have enough memory, because the operating system caches recently used disk blocks in memory anyway. Rather, they can be faster because they can avoid the overheads of encoding in-memory data structures in a form that can be written to disk.

Besides performance, another interesting area for in-memory databases is providing data models that are difficult to implement with disk-based indexes. For example, Redis offers a database-like interface to various data structures such as priority queues and sets. Because it keeps all data in memory, its implementation is comparatively simple.




