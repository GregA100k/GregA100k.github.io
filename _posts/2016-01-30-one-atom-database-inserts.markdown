---
layout: post
title:  "One Atom Database: Inserts and Updates"
date:   2016-01-30 18:00:00
categories: clojure 
tags: clojure oneatomdb
---
oneatomdb So Far
================

The [previous post]({% post_url 2016-01-22-one-atom-database-join-keys %})
described prefixing the columns with the list name as the fix for joining 
two lists that had maps with the same keys.

>(seethe @db2 ["jointhe" :runners :laps "onthe" :runners.racenumber = :laps.runnernumber] wherethe :runners.racenumber = "3")

Joining multiple topics together was one of the last big pieces for 
retrieving data, so at this point, oneatomdb works as a read only database,
but there's no way to put data into it.  Inserts and updates are needed.  

Inserts
=======
Insert logic can be broken down from the inside out.  The row that is 
being inserted is really a map.  That row is being added to a
vector.  This can be done individually with conj or over a list using
apply conj.  Since that vector is the value that goes with a top level
key in the atom database, it needs to be associated with that key in
the map.  Finally, that association is swapped into the atom.

{% highlight clojure %}
(defn insertthe [a topic newval]
  (if (vector? newval)
  (swap! a assoc topic (apply conj (topic @a) newval))
  (swap! a assoc topic (conj (topic @a) newval))
))
{% endhighlight %}

Updates
=======
Update is a more interesting operation.  Again, starting at the innermost
layer, an update assicates a new value with a key in a map.  That updated
map is then replaced inside a the vector which is then replaced as the 
value associated with a top level key in the atom database.

Each of those operations can be done with the 
[assoc-in](http://clojuredocs.org/clojure.core/assoc-in) function.  Assoc-in 
for a vector uses the index of the element of the vector as the key.  That's
great if you know which entry is to be replaced.  As part of an update 
command, any value that passes a predicate should be updated.  Map can be
used to look at each element in the vector, but that doesn't give the index
so my initial implementation mapped over the keys of the vector.

>(range (count v))

Each map in the vector is checked against the wherethe function and if 
it is true, that map is updated using assoc-in.  


{% highlight clojure %}
(defn updatethe [a topic setthe column newval & filterlist]
  (doall (map (fn [i] 
        (let [r (get (topic @a) i)
              mes (println "  " r)]
         (if ((apply wherethe (rest filterlist)) r)
           (swap! a assoc-in [topic i column] newval))
       )) (range (count (topic @a)))))
)
{% endhighlight %}


The initial implementation worked, but there is a 
[map-indexed](http://clojuredocs.org/clojure.core/map-indexed)
function which provides the index. 

{% highlight clojure %}
(defn updatethe [a topic setthe column newval & filterlist]
  (let [pred (apply wherethe (rest filterlist))
        updatefun (fn [idx itm]
                   (if (pred itm)
                     (swap! a assoc-in [topic idx column] newval)))
       ]
       (doall (map-indexed updatefun (topic @a)))))
{% endhighlight %}

Since both map and map-indexed return a lazy sequence, and updating the
atom is a side effect, the update won't happen unless 
[doall](http://clojuredocs.org/clojure.core/doall) is used to force evaluation
of the whole collection.

Current State of oneatomdb
==========================
At this point, oneatomdb can add, update, and retrieve data.  While it's 
not complete, it may be close enough to try it in a test app.

What's Next
===========
There are three potential next steps with oneatomdb.  One choice is to
implement the 'D' operation in CRUD and add a deletethe function.
Another choice is to write the reference app to use oneatomdb, and the 
last option is to clean up some of the inconsistencies in the code.

During the development of the seethe function, the first argument has been
the de-referenced map out of the atom.  Insert and update operations 
actually need the atom so insertthe and updatethe are inconsistent from 
seethe. 

There is also a cleanup of places in the api where a string,
like "wherethe", is passed that just gets mapped to a function.  Converting
wherethe into a function should remove some cruft.
