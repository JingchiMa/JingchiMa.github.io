---
title: Log-based Storage
categories:
  - Distributed System
tags:
  - dda
---

## Partition on Primary Key
- Partitioning by key range
  - support range queries
  - may cause hot spot
- Partitioning by hash range
  - more even load
  - do not support range queries

{% capture notice-2 %}
**A Compromise in Cassandra**  
Cassandra takes a compromise between the two approaches by using *compound primary key*. The hash of the first part will be used to partition the data, and within each partition the other parts will be used as a concatenated index for sorting the data in SSTable.

For example, a table contains users' posts can use (user_id, update_time) as primary key. And one can quickly determine which partition contains the data for a certain user, and search the updates in a time range efficiently.
{% endcapture %}

<div class="notice--info">{{ notice-2 | markdownify }}</div>

## Partition and Secondary Index
### Document-based partitioning
- **local index**: each partition maintains its own secondary indexes, covering only the documents in this partition.
- fast for write because write only touches one partition.
- read can be slow because you may search all the partition and merge the results.
- This technique is also known as "scatter and gather"
### Term-based partitioning
- **global index**: partition the indexes based on the secondary indexes.
  For example, a second index is color, and we have built the indexes. Then we can use either key-range partitioning or hash-range partitioning to store the results to the partitions.
- fast for read: can directly find which partition to go to
- slow and complicated for write: need distributed transaction across all the partitions affected.
  
