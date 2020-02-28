---
title: Log-based Storage
categories:
  - Distributed System
tags:
  - on-disk data structure
  - storage
  - dda
---

## Data Structure
Two major components:
- on disk append-only file.
  The file contains the actual key-value pair.
- in-memory hash index
  A hashmap maps from key to byte-offset in the file.

## Operations
### Read
To read a key, first check the in-memory hashmap and find the offset, then do a disk seek and retrieve the value.

### Write / Update
To write / update (k, v), append the entry to the end of the file and update the in-memory hashmap with the new offset.

### Delete
A special identifier (sometimes called *tombstone*) will be appended to denote the certain key is removed, and the hashmap is also updated.

## Comments
### Performance
Read and write are both pretty fast. A write only need one disk I/O.

### Compaction
For garbage collection, there is usually a maximum size for each segment(file). After reaching the limit, compaction will start and multiple segments will be merged together. For keys with multiple values, the most recent value will be kept. After compaction, new segment will be created and old ones can be deleted.

### Problems
- cannot support range queries on keys.
- all the keys must fit into memory.
