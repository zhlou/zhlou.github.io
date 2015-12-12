---
layout: post
title: "Finding XML Root Element Using Boost Property Tree"
categories: c++ boost property_tree XML
date: 2015-12-12
---
Recently, for [various reasons][libxml-post], I need to convert some XML parsing code from using `libxml2` to `boost::property_tree::ptree`. During such migration, one issue arises from the different way two libraries treating the root element of a XML document.

In `libxml2`, one is able to get the root element without knowing its name by using the function `xmlDocGetRootElement()`. Being spoiled by this feature, our XML files grow a little messier than they otherwise should have and having multiple different names for the root element. In order to be able to process these files, we need the new code to be able to replicate the old behavior.

This requirement creates some difficulties as `ptree::get_child()` requires the name of each node along the path. To be honest, `boost::property_tree` is a general purpose tree structure that supports not only XML but also JSON and INI format. As a result, its XML parser is much more tolerant than a pure XML parser. For example, it will be perfectly happy to parse a file with multiple root elements which most other XML library will complain about. As a side effect, the root element in `ptree` does not have special status.

This means I need to write my own function that finds the root element without referring to its name. Fortunately, `ptree` provide a Container-like interface that allow me to iterate through the (key-node) pairs of any node's direct sub-nodes. The pair is defined as

```c++
typedef std::pair<const Key, self_type> value_type;
```

where `self_type` is a `typedef` of the node itself. Knowing this, I can simply get the first sub-node from the container. The code snippet looks like the following:

```c++
#include <boost/property_tree/xml_parser.hpp>
#include <boost/property_tree/ptree.hpp>
using boost::property_tree::ptree;

std::string filename("some_file_name");
ptree pt;
read_xml(filename, pt, boost::property_tree::xml_parser::trim_whitespace);
ptree &root = pt.begin()->second;
```

The `second` used here because `pt.begin()` returns an iterator that dereference to an `std::pair` where `first` is the node name and `second` is the sub-node itself.

Certainly, this code gives you the root element if the XML file is well-formed. If there are multiple root elements in the XML document, this will return the first one. If there is no root element, it will throw an exception as that will cause to dereference a past-the-end iterator.


[libxml-post]: {% post_url 2015-12-05-using_libxml2_on_supercomputer_considered_harmful %}
