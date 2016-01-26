---
layout: post
title: "Notes on Hash Table"
categories: algorithm
---
This post is my study notes on the algorithms and implementations of hash tables.

## Hash Functions
### Hash Functions for Strings
A common hash function for strings looks like the following:

```c
unsigned hash(char *str) {
    unsigned h = M; // initial value
    while(str) {
        h = h * C + str; // implicit mod 2**(sizeof(unsigned)*8)
        ++ str;
    }
    return str;
}
```
In particular, the famous [`djb2`][djb2] hash function by Danial J. Bernstein uses `M=5381` and `C=33`. The multiplication by `33` or other value in the form of `2**n + 1` can be done by

```c
h = (h << 5 + h) + str; // where n = 5 => C = 33
```
Some argue that multiplication in modern CPUs is fast enough so this is no longer necessary.

## Collision Resolution
Collision happens when different keys generate the same hash. The two major methods for collision resolution are separate chaining and open addressing.

### Separate Chaining
Separate chaining is conceptually simple: each entry in the hash table is a list or list-like structure (e.g. head cells) so all keys hash to same index are stored in the same list. Adding and deleting keys are simply the list operations on the corresponding list.

The major downside of separate chaining is the peformance as inserting new entry into the list involves allocating new memory. Following the list may also cause all sorts of performance issue due to non-locality in memory.

### Open Addressing
Open addressing store all the entries in the bucket array itself. When collision happens, it try to find an unoccupied slot according to some "probe sequence". The choice of the probe sequences include:

* linear probing: simply trying the next slot
* quadratic probing: index is calculated using some quadratic polynomial (python uses this method)
* double hashing: next index is calculated by another hash function

One chanlleage in open addressing is when deleting entries, setting the entry back to unoccupied state will break the hash table if there are more entries of the same hash. A common strategy is to set the entry to some dummy state that is different from empty.

## Table Size and Dyanmaic Resizing

## Reading Materials

* [Chapter 3.4 of Algorithms.][algorithms-hash]
* [Wikipedia page about Hash Table.][wiki-hash]
* [cPython's dictionary implementation.][python-dict]

[djb2]: http://www.cse.yorku.ca/~oz/hash.html
[algorithms-hash]: http://algs4.cs.princeton.edu/34hash/
[wiki-hash]: https://en.wikipedia.org/wiki/Hash_table
[python-dict]: http://www.laurentluce.com/posts/python-dictionary-implementation/
