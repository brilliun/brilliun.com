---
title: "The InnoDB Storage Engine"
date: 2018-04-14T22:23:45+09:00
lastmod: 2018-04-14T22:23:45+09:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

<!-- toc -->

Starting from MySQL 5.5.5, the default storage engine for new tables is InnoDB rather than MyISAM.

Multi-Version Concurrency Control (MVCC)
=================================
This technique lets InnoDB **transactions** with certain **isolation levels** perform **consistent read** operations; that is, to query rows that are being updated by other transactions, and see the values from before those updates occurred. This is a powerful technique to **increase concurrency**, by allowing queries to proceed without waiting due to **locks** held by the other transactions.

With MVCC, InnoDB keeps information about old versions of changed rows, to support transactional features such as concurrency and rollback. This information is stored in the tablespace in a data structure called a *rollback segment*. InnoDB uses the information in the rollback segment to perform the undo operations needed in a transaction rollback, or build earlier versions of a row for a consistent read.


InnoDB Locking
=================
## Shared and Exclusive Locks

Both are row-level locking.

* A shared (`S`) lock permits the transaction that holds the lock to read a row.
* An exclusive (`X`) lock permits the transaction that holds the lock to update or delete a row.

### Case 1: Someone holding S lock

If transaction `T1` holds a shared (`S`) lock on row `r`, then requests from some distinct transaction `T2` for a lock on row `r` are handled as follows:

* A request by `T2` for an `S` lock can be granted immediately. As a result, both `T1` and `T2` hold an `S` lock on `r`.
* A request by `T2` for an `X` lock **cannot** be granted immediately.

### Case 2: Someone holding X lock

If a transaction `T1` holds an exclusive (`X`) lock on row `r`, a request from some distinct transaction `T2` for a lock of **either type** on `r` **cannot** be granted immediately. Instead, transaction `T2` has to wait for transaction `T1` to release its lock on row r.

## Intention Locks

InnoDB supports coexistence of row-level locks and locks on entire tables. Intention locks make it possible.
Intention locks are table-level locks in InnoDB that indicate which type of lock (shared or exclusive) a transaction requires later for a row in that table. There are two types of intention locks:

* Intention shared (`IS`): Transaction `T` intends to set `S` locks on individual rows in table `t`.
* Intention exclusive (`IX`): Transaction `T` intends to set `X` locks on those rows.

The intention locking protocol is as follows:

* Before a transaction can acquire an `S` lock on a row in table `t`, it must first acquire an `IS` or stronger lock on `t`.
* Before a transaction can acquire an `X` lock on a row, it must first acquire an `IX` lock on `t`.

Consistent Reads
================
A read operation that uses **snapshot** information to present query results based on a point in time, regardless of changes performed by other transactions running at the same time. If queried data has been changed by another transaction, the original data is reconstructed based on the contents of the **undo log**. This technique avoids some of the **locking issues** that can reduce **concurrency** by forcing transactions to wait for other transactions to finish.

With **REPEATABLE READ isolation level**, the snapshot is based on the time when the first read operation is performed. With **READ COMMITTED** isolation level, the snapshot is reset to the time of each consistent read operation.

Consistent read is the default mode in which InnoDB processes SELECT statements in **READ COMMITTED** and **REPEATABLE READ** isolation levels. Because a consistent read does not set any locks on the tables it accesses, other sessions are free to modify those tables while a consistent read is being performed on the table.

