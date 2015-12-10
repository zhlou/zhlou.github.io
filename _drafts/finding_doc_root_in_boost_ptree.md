---
layout: post
title: "Finding XML Root Element Using Boost Property Tree"
categories: c++ boost xml ptree
---
Recently, for [various reasons][libxml-post], I need to convert some XML parsing code from using `libxml2` to `boost::property_tree::ptree`. During such migration, one issue arises from the different way two libraries treating the root element of a XML document. In `libxml2`, one can get the root element without knowing its name by calling
{% highlight c++ %}
xmlNodePtr root = xmlDocGetRootElement(doc);
{% endhighlight %}
On the other hand, `ptree::get_child()` requires the name of each node along the path.

[libxml-post]: {% post_url 2015-12-05-using_libxml2_on_supercomputer_considered_harmful %}
