---
title: SSTable and Lookup in LevelDB
date: 2017-03-26 08:19
author: selcuk
image: 
  thumbnail: /assets/images/sstable.png
---
The very basic idea behind LevelDB and BigTable of Google is Log-Structure Merge tree and Sorted String Table (SSTable). Generally LSM tree use a log to trace all index hence it has better read performance while trade off write performance. From bLSM (reference spot), we can easily spot the difference on design principles between LSM tree and B-tree.

![sstable](/assets/images/sstable.png)

Another important data structure in LevelDB is Sorted String Table (SST). A sorted table stores a sequence of entries sorted by key. Each entry is either a value of this key nor a deletion marker of this key.

Usually, when LevelDB performs one LookUp, it would firstly look for the key range in Manifest file. And below is the detailed work flow of one LookUp in levelDB:
<ol>
 	<li>A Manifest file lists the set of sorted tables that make up each level, the corresponding key ranges, and other important metadata.</li>
 	<li>After know which SST to look up. LevelDB would simple open a specific SST and load the index part, at the end of each SST, into memory.</li>
 	<li>Perform a B<strong>inary Search</strong> on the key range in this SST.</li>
 	<li>After the confirmation of a specific key in this SST, it would look further in the internal data structure of SST - data block.</li>
 	<li>Among each data block, there are a few restart points to record the key range of each block.</li>
 	<li>LevelDB would do a linear search on a specific data block with effective key range of a specific key.</li>
</ol>
A structured view of SST:
<pre><code>&lt;beginning_of_file&gt;
[data block 1]
[data block 2]
...
[data block N]
[meta block 1]
...
[meta block K]
[metaindex block]
[index block]
[Footer]        (fixed size; starts at file_size - sizeof(Footer))
&lt;end_of_file&gt;</code></pre>
