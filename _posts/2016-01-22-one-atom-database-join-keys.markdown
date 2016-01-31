---
layout: post
title:  "One Atom Database: Join Keys"
date:   2016-01-22 18:00:00
categories: clojure 
tags: clojure oneatomdb
---
oneatomdb So Far
================
The [previous post]({% post_url 2015-12-29-one-atom-database-joins %})
allowed querying accross multiple top level key lists in the one atom.

>(seethe @app-state ["jointhe" :runners :laps "onthe" :racenumber = :runnernumber :courses "onthe" :course = :id] wherethe :racenumber = "3")))

The resulting map has key value pairs from all the component maps.  A
problem occurs when two maps have keys with the same name. The last one key
value pair added to the result replaces the earlier pair.  The solution is 
to prefix the keys in the result map with the key for list.

{% highlight clojure %}
(defn build-join-name [mapname columnname]
  (keyword (str (name mapname) "." (name columnname))))
{% endhighlight %}

The downside of prefixing the values in the keys in the result is those
prefixes are also needed in the wherethe and jointhe conditions.

Current State of oneatomdb
==========================

At this point oneatomdb is essentially the same as in the last post.
Filters and join conditions can be applied.  Join conditions are 
limited to a single comparison at this time but the problem when joined 
maps have keys in common has been fixed..

>(seethe @db2 ["jointhe" :runners :laps "onthe" :runners.racenumber = :laps.runnernumber] wherethe :runners.racenumber = "3")

What's Next
===========
[Inserts and updates.]({% post_url 2016-01-30-one-atom-database-inserts %})
