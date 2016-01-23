---
layout: post
title:  "One Atom Database: Joins"
date:   2015-12-29 18:00:00
categories: clojure 
tags: clojure oneatomdb
---
oneatomdb So Far
================
The [previous post]({% post_url 2015-12-23-one-atom-database-combining-operators %})
expanded on oneatomdb which is a library which can query a list of items 
out of the one atom that is like a database for an om app.  Queries can 
be filtered with combinations filters joined with 'and' and 'or' operations.

>(seethefun @app-state :runners)

>(seethefun @app-state :runners "wherethe" ["orthe" :racenumber = "4" :racenumber = "5"])

Joining
=======
In the application that spawned this project, I was keeping track of runners
in a multi-lap trail race so before the race I had a list of runners and a 
list of courses.  Once the race started, the application built up a list of 
laps which had references to the runner and the course along with the 
time that the lap was completed. Below are example maps of each type.

>Runner: {:firstname "Greg" :lastname "Allen" :racenumber 3}

>Course: {:name "Long Loop" :label "l" :distance 3.35}

>Lap: {:runnernumber 3 :course "l" :elapsed-time 20778}

To build the results for a single runner, information from all three maps is 
needed.

>(seethefun @app-state :runners "wherethe ... )

needs to be changed to include the join 

>(seethefun @app-state ["jointhe" :runners :laps "onthe" :racenumber = :runnernumber :courses "onthe" :course = :label] wherethe ... )

The api that I have chosen follows the pattern used for wherethe conditions. A
vector starting with the "jointhe" identifier followed by the two groups of
data being joined and then the join condition.  While doing the join, the first
set of information can either be a keyword which maps to the name of the 
top level key in the map, or a map itself.  Passing a map allows for the 
result of the first join to be passed to subsequent joins.

Joining data from two maps is a pretty straight forward process if it is 
constrained and if it is done naively.  Each map that passes the join 
condition test is merged together.  

{% highlight clojure %}
(for [a firstmap
        b secondmap
        :when (joincondition a b)]
        (merge a b))
{% endhighlight %}

This requires building a function that tells if the join conditions 
are met.  The initial implementation constrains each join to a single join 
condition.

{% highlight clojure %}
(defn build-join-condition [f1 comp f2]
  (fn [m1 m2] (comp (f1 m1) (f2 m2))))
{% endhighlight %}

Just merging the two maps is, in general, too simple of a plan as well.  If
both maps have keys that match there will be data lost.  In my example, if 
runners had a single :name field instead of separate first and last name
fields, then joining runners, laps, and courses would end up with the
:name key pointing to "Long Loop" and the runner's name would be lost.

Current State of oneatomdb
==========================

The queries that oneatomdb can handle are getting more useful.  Filters and
join conditions can be applied.  Join conditions are limited to a single
comparison at this time and there is a problem if the joined maps have
any keys in common.

What's Next
===========
Either multiple join conditions should be allowed or [fixing the common key
problem]({% post_url 2016-01-22-one-atom-database-join-keys %}).
