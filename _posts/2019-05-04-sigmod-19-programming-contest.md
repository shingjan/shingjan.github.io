---
title: SIGMOD 2019 Programming Contest - Parallel Radix Sort
date: 2019-05-04 11:59
author: selcuk
image: 
  thumbnail: /assets/images/sigmod19.jpg
---

This post intends to make a note of how Zhaoxing and I competed in the programming contest of ACM SIGMOD conference 2019, which is one of the top and premium venues for database-related publications. The contest is first hosted at SIGMOD 2009 and is made annual afterwards. We made the finalist list of this year and won a prize award to present our design and implementation of Leyenda at Amsterdam. The following will be a brief break-down and summary.

## Setup

The topic of this year's programming contest is sorting. Contestants are asked to sort **10/20/60** GB of file on the testbed machine. Initially it was something like 10/20/40/100 but sorting 100 Gb of file on the testbed takes too long for any team to push a last-minute change so they decided to increase the third large dataset to 60 with skewed data and eliminate the forth one. For the testbed, there are roughly 30 Gb of memory and 20 cores with 40 hyperthreads. Seems normal, right? The thing we weren't told is that the devil is in the detail as all memory actually resides on **NUMA** node 0, which means threads on other nodes will have a longer access latency to memory and becomes our own kind of *Achilles' Heel* when every millisecond counts.

## Radix Sort

**Radix sort** is a round-based non-comparative sorting algorithm that iterates through sorting keys in digits to separate them into buckets, grouping records by the individual digits, which share the same position and value. For example, records “abc”, “bcd”, “cde”, “cfe”, and “dfg” can be separated into four buckets according to the first character “a”, “b”, “c”, and“d” in the first round of radix sort. In the upcoming rounds, buckets will be further divided until all records are in sorted order. Each round of radix sort is of **O(n)** time complexity, and there can be at most **w** rounds, with **w** being the word length of the sorting key, so the overall time complexity of radix sort algorithm is **O(w * n)**. Radix sort performs better than comparative sorting algorithms, like QuickSort or MergeSort,on workloads with fixed-size keys on each record. They are variations like least-significant digit radix sort and most-significant digit radix sort for varied data distributions. 

## Key Points 

> TL;DR
- NUMA awareness
- Page Cache 
- In-house implementation of Radix Sort

For small and medium, an important observation is that one doesn't need to write a single byte to the disk. There is enough memory to just write directly to the page cache, which is much faster. On large it's probably more complex, but a vanilla external sort with merging of intermediate results from disk could get pretty decent results assuming one overlay reading input with sorting the intermediate results and store the last intermediate result in memory without writing to disk. 

The same idea applies for small and medium, every byte saved in memory reduces the I/O cost, so aggressively unmapping/freeing allocated memory at the moment it's no longer needed, on a small page granularity, is a good idea. It is also worth it to write the intermediate results to disk from the end to start , in the opposite order, so that the last result stays in the page cache when you start to merge assuming you read it from the beginning. 

For more general stuff, huge pages for large buffers might help and NUMA awareness is also important - pinning all threads to NUMA node 0, where all the memory resides, can provide a huge improvement on small/medium, if there is no other use for the threads, otherwise they can hurt the memory bandwidth. We also ended up writing our own radix sorting implementation which brought us some decent advantage on small and medium workloads.

## In a Nutshell 

Some points are quoted from our discussion forum as all implementations from the finalist teams share some similarities. And big thanks to Zhaoxing, without whom our two-people team can never make it to the finalist list. Even though one of the professors, who wasn't much help, thought our staff is not "research" enough to be a senior-level CS grad class project, we nailed it anyway. And it is interesting to see many renowned research projects are spin-offs from programming contests, like Apache Spark, AlexNet and Heterogeneous Parallel Virtual Machine. It's been fun. Period.