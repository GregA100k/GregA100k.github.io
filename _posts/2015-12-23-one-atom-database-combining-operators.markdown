---
layout: post
title:  "One Atom Database: Combining Operators"
date:   2015-12-23 18:00:00
categories: clojure 
tags: clojure oneatomdb
---
oneatomdb So Far
================
The [previous post]({% post_url 2015-11-16-one-atom-database %})
introduced oneatomdb which  can read a list of items out of the 
one atom that is like a database for an om app.  It can also filter that list

>(seethe @app-state runners)

>(seethe @app-state runners wherethe lastname = Allen andthe firstname = Greg)

Filtering with 'or'
===================
Expanding the functionality to include 'or' conditions taught me two things.
First, parsing sql strings is something that I don't want to do and, second, 
recursively applying the filters gets complicated when conditions are not just
'and'ed together.

Part of the reason for starting with the Adam 12 themed 'seethe' rather
than using 'select' to query was to avoid falling into the trap of trying 
to completely re-implement sql.  This made it easier to replace the query 
filter syntax with  more of a lisp syntax which simplifies the parsing.

> (seethefun @dbmap :runners "wherethe" ["orthe" :racenumber = 3 :racenumber = 4])

This example shows the function version of seethe.  In the function version,
the column names are being passed as keywords rather than strings and the 
strings are being passed in quotes.  This eliminates some of the macro magic
that made the query read more like text than code.  A bonus that falls out 
from this is that functions can be used in place of keywords in the query, 
so if there is a fullname function which composes the first and last 
name together into a string,

>(defn fullname [m] (str (:firstname m) " " (:lastname m)))

That fullname function can be passed into a query.

>(seethefun @dbmap :runners "wherethe" fullname = "Greg Allen")

Composable Filters
==================

With a more manageable syntax for querying, it was easier to rewrite the 
filtering process.  This rewrite turned out to be a real step into the 
world of functional programming for me.  Writing a function to handle 
any combination of filters seemed really hard, but writing a function 
to build a function that can handle any combination of filters was much easier.  
Starting with a simple function that builds a comparison function for a single
operation.

{% highlight clojure %}
(defn build-compare [field-name comparison field-value]
  (fn [m]
    (comparison field-value (field-name m))))
{% endhighlight %}

This handles a simple case of only one filtering function. Multiple filtering
functions are combined into a list of comparison functions.  Nested 
comparison functions are built up recursively.

{% highlight clojure %}
(defn build-comparison-list [& filterlist]
  (cond (empty? filterlist) []
        (coll? (first filterlist)) (conj 
                                     (apply build-comparison-list (rest filterlist)) 
                                     (apply wherethe (first filterlist)))
        :else (let [[field-name comparison field-value & therest] filterlist]
                (conj 
                  (apply build-comparison-list therest) 
                  (build-comparison field-name comparison field-value)))
  ))

{% endhighlight %}

The list of filtering functions is passed to every? for 'andthe' filters 
and to some for 'orthe' filters.

{% highlight clojure %}
(defn andthe [& args]
  (fn [m] (every? #(% m) (apply build-comparison-list args))
  ))

(defn orthe [& args]
  (fn [m] (if (some #(% m) (apply build-comparison-list args))
            true
            false)
))
{% endhighlight %}

Current State of oneatomdb
==========================
Queries can now be used against a list of maps.  Query filters can be built
up of individual comparisons 'and'ed and 'or'ed together.  The filters can
be nested to build up complex queries.

What's Next
===========
Joining data from different lists of maps.
