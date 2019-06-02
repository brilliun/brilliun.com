---
title: "Implementation of API Pagination"
date: 2018-06-14T22:20:53+09:00
lastmod: 2018-06-14T22:20:53+09:00

description: ""
keywords: []
tags: []
categories: []
author: ""
draft: false
isCJKLanguage: false

# thumbnailImage: image-1.png
# thumbnailImagePosition: bottom
# autoThumbnailImage: yes

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true

---

<!-- toc -->

# Overview

Stable pagination is not a silver bullet. Consider sites like Reddit and Hacker News. Their pagination is inherently unstable: items are sorted by a volatile ranking, frequently moving up and down the list over time. How do they deal with it? They don't.

Both sites frequently showcases how bad it can look by showing duplicate items as you advance pages. It's not all that bad, because neither site subscribes to the infinite scrolling crowd (yet?).

However, there are plenty of client apps out there that present both Reddit and Hacker News as an infinite scrolling feed; and some of them just show duplicate items stacked on top of each other as you scroll down. Amateur hour! All they need to do to fix that half of the problem is to ignore duplicate items when appending the next page.

# Offset-based Pagination (Classic)

Using offsets to paginate data is one of the most widely used techniques for pagination. Clients provide a parameter indicating the number of results they want per page, count, and a parameter indicating the page number they want to return results for, page. The server then uses these parameters to calculate the results for the specific page being requested. Implementing this with an SQL-like database is very straight forward.

## Advantages

* It gives the user the ability to see the total number of pages and their progress through that total.
* It gives the user the ability to jump to a specific page within the set.
* It’s easy to implement as long as there is an explicit ordering of the results from a query.

## Drawbacks

* Slow. An issue here is the OFFSET clause that is used in the server’s database query. It’s incredibly slow when it comes to tables with millions of entries. There is no index involved that may speed up our queries. 

  * Using LIMIT <count> OFFSET <offset> doesn’t scale well for large datasets. As the offset increases the farther you go within the dataset, the database still has to read up to offset + count rows from disk, before discarding the offset and only returning count rows.

* Error-prone: It’s really easy to miss elements if the element order changes during two requests. If an element of a previous request is deleted, one element of the following page moves up to a page that has already been delivered.


# Timestamp-based Pagination

More general version is Keyset-based pagination.

## Advantages

* Works with existing filters without additional backend logic. Only need an additional limit URL parameter.
* Consistent ordering even when newer items are inserted into the table. Works well when sorting by most recent first.
* Consistent performance even with large offsets.

## Drawbacks

* Endless loops. We can end up in endless loops if all elements of a page have the same timestamp. In practice, this can easily happen after a bulk update of many elements. The only thing we can do here is to make endless loops less likely by:
* Not efficient. The client may receive and process the same elements multiple times when many elements have the same timestamp and they are overlapping two pages.
* Tight coupling of paging mechanism to filters and sorting. Forces API users to add filters even if no filters are intended.
* Does not work for low cardinality fields such as enum strings.
* Complicated for API users when using custom sort_by fields as the client needs to adjust the filter based on the field used for sorting.


# Cursor-based Pagination

Cursor-based pagination works by returning a pointer to a specific item in the dataset. On subsequent requests, the server returns results starting from the given pointer. This method addresses the drawbacks of using offset pagination, but does so by making certain trade offs:

* The cursor must be based on a unique, sequential column (or columns) in the source table.
* There is no concept of the total number of pages or results in the set.
* The client can’t jump to a specific page.

## Custom Sort Order

When need custom sort order, there are two approaches:

* Only enclose id into cursor, and execute a query first to get the value of sort field for record#id
* Enclose the custom sort field value into cursor, too. (we may use this approach)


# Conclusion

In offset pagination, we can sort by any column and paginate the results while cursor based pagination depends on the sorting of the unique cursor column.

Offset pagination contains page numbers in addition to next and previous links. But due to the highly dynamic nature of the data, we can’t provide page numbers for cursor based pagination.

Generally, offset pagination allows us to navigate in both directions while cursor based pagination is mostly used for forward navigation.


![Part1](/images/api-pagination/part1.png)
![Part2](/images/api-pagination/part2.png)


# Sample Response

```
{
  "data": [...]
  "meta": {
    "pagination": {
      "limit": 25,
      "next_fetch": "https://api.example.com/api/v1/missions/:uuid/parameters?limit=25&cursor=aiufqwe92rjkdxfasf",  // null if no more data
      "next_cursor": "aiufqwe92rjkdxfasf"  // null if no more data
    }
  }
}
```

## Encoded Cursor

We will use Base64 to encode information on how to retrieve the next set of results within the cursor. This allows us to technically implement different underlying pagination schemes per endpoint, while giving a consistent interface to consumers of the API.


# Reference

* https://www.sitepoint.com/paginating-real-time-data-cursor-based-pagination/
* https://developer.twitter.com/en/docs/tweets/timelines/guides/working-with-timelines
* https://slack.engineering/evolving-api-pagination-at-slack-1c1f644f8e12

