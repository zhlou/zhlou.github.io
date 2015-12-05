---
layout: post
title: "Using libxml2 on Supercomputers Considered Harmful"
categories: c++ libxml2 libxml Cray XML
date: 2015-12-05
---

I was blissfully unaware of a nasty bug in `libxml2` and happily using it for my parallel simulated annealing code running on [Beagle][beagle-website]. That is, until last week when Beagle underwent a system upgrade and all of sudden my code start segfaulting. Upon inspecting the core dump, I was surprised to find out that it is `xmlInitParser`, which is in turn called by `xmlParseFile` that was causing the segmentation fault.

Since Beagle uses different systems for its head nodes and compute nodes, like many supercomputer systems do, my first intuition was that the libxml2 from the head nodes might be no longer work on compute nodes. If that was the case, simply compiling `libxml2` from the source using the cross compiler should fix that. Unfortunately, it was not that simple. A freshly compiled `libxml2` still causes segfaults.

A few googles later I landed at [this bugzilla page][libxml2-bugzilla]. Basically  `libxml2` implicitly uses pthreads and somehow statically linking to it might cause the the code to segfault. Even worse, the maintainer refused to fix this bug citing "[Static Linking Considered Harmful][no-static-linking]".

It is this attitude, not the bug, that made me furious. Surely I understand it is usually a bad practice to statically link some library. But for many supercomputers, statically linking the library is the only viable option. By refusing to fix this bug, the fine people at Gnome simply ignored the constraint we are having in the scientific computing community.

In an ideal world where the bandwidth, IO, and storage are all infinitely abundant, certainly linking library dynamically would be the correct way to do things. However, many current supercomputers, like IBM BlueGene series or Cray XE series (on which Beagle is based) or newer, uses severely limited compute nodes in order to achieve maximal performance. These "nodes" usually don't even have their own storage beyond the stripped down kernel in their firmware. Famously, the original IBM BlueGene/L model doesn't even support `fork` or `system` calls on the compute node. The only real filesystem they can access is the network shared user filesystem. Hence, it is common practice for the vender to supply compilers that only produces statically linked binary.

That left me with little choice. Surely I can go down the rabbit hole and try to patch the libxml2 code myself, provided the patch attached in the ticket is still relevant 5 years later. But I guess that won't be the best use of my time. Since their attitude left me a bitter taste, I'll declare that using `libxml2` on supercomputers considered harmful, and switch my XML parsing code entirely to `boost::property_tree`. Yes, the code I'm working on contains both C and C++ code. Actually, it contains decades of work by multiple people but that will be a post for another day.


[beagle-website]: http://beagle.ci.uchicago.edu
[libxml2-bugzilla]: https://bugzilla.gnome.org/show_bug.cgi?id=609926
[no-static-linking]: http://www.akkadia.org/drepper/no_static_linking.html
