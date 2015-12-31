---
layout: post
title:  "One Atom Database"
date:   2015-11-16 22:00:00
categories: clojure 
tags: clojure oneatomdb
---

This post will describe the beginnings of oneatomdb
which is a database like querying tool for maps like the app-state
atom used in the [Basic Tutorial](https://github.com/omcljs/om/wiki/Basic-Tutorial) for om.

There are a few assumptions that this tool is based on.  First is the top
level keys are keywords.  Given a simple enough query, that should be the 
only constraint.  For more complex queries, the value of the key should
be a list of maps.

>One Adam 12. One Adam 12. See the burglary in progress at ...

The kids in my neighborhood used to watch Adam 12 and then go play it outside.
We'd run around with sticks, playing cops and robbers, repeating the radio
call "One Adam 12. One Adam 12" over and over.  That was was the first 
thing I thought of when the thought of having one atom hold the application 
state for an om application.

The API
-------
To start with, the Adam 12 influence is in the forefront of the api.
'seethe' will be the starting point to query the application state. 
In addition to making me smile each time I make a call, I won't be as 
tempted to implement SQL as I would be if I used 'select'

The first design decision was how to 'see the' application data.  
I had a choice of trying to tie in to om somehow, or just pass the 
@app-data into the oneatomdb functions. I chose to pass in the map.

Starting as simply as possible, I need a query to bring back the 
value of a top level key.  
In my case, I had a list of runners.  
If I used a function, it had to be either 

>(seethe @app-state :runners)

or 

>(seethe @app-state "runners")

For a cleaner api, I chose to use a macro so I could do

>(seethe @app-state runners)


{% highlight clojure %}
(defmacro seethe [db-map topic]
  (list (keyword topic) db-map))
{% endhighlight %}

The next step is to filter the results like with a where clause in a sql query.
To be consistent with the seethe name, I'll use a wherethe parameter. 
For example:

>(seethe @app-state runners wherethe race-number is 3)

{% highlight clojure %}
    (defmacro seethe [db-map topic & [_ field-name _ field-value :as args]]
      (if args 
        `(filter #(= ~(str field-value) (~(keyword field-name) %)) (~(keyword topic) ~db-map))
        `(~(keyword topic) ~db-map)))
{% endhighlight %}

At this point, the code is pretty simple. There are only two possibilities: 
either there's a where clause or there isn't and the two are handled separately
by the two branches of the if statement. 

At this point, only one where clause is handled and the comparison is 
hard coded to equals.  Passing in the comparison operator feels like it 
will be an easier change so I'll handle that one next.


{% highlight clojure %}
    (defmacro seethe [db-map topic & [_ field-name comparison field-value :as args]]
      (if args 
        `(filter #(~comparison ~(str field-value) (~(keyword field-name) %)) (~(keyword topic) ~db-map))
        `(~(keyword topic) ~db-map)
  ))
{% endhighlight %}

Naming the comparison function parameter and passing a function takes care of
the hard coded equals.

Next up is to have more than one filter. Again, there is a choice to be made.
Multiple 'and' operations seems easier to add than a combination of 'and' and
'or'.  Multiple 'and' filters can be done with recursion. 

Doing recursion with a macro is possible, but my attempt at it brought up 
one of the constraints of using macros.  Macros are not functions so it
is not possible to apply the macro to a list of arguments.

Although there is probably a cleaner way to do this, here is what I have
working.
{% highlight clojure %}
(defmacro seethe
  ([db-map topic] `(~(keyword topic) ~db-map))
  ([db-map topic filter-list]
    (let [[operator field-name comparison field-value & therest] filter-list]
      (if operator
        `(filter #(~comparison ~(str field-value) (~(keyword field-name) %))
           (seethe ~db-map ~topic ~therest)
         )
      `(~(keyword topic) ~db-map))))
  ([db-map topic operator field-name comparison field-value & therest]
    (if therest
       `(filter #(~comparison ~(str field-value) (~(keyword field-name) %))
          (seethe ~db-map ~topic ~therest)
        )
       `(filter #(~comparison ~(str field-value) (~(keyword field-name) %))
          (~(keyword topic) ~db-map)
        )
    )))

{% endhighlight %}

For now, the most important thing that I have learned is that it's hard
for me to do development using macros.  I spend as much or more time trying
to get the macro to work as I do trying to code the feature I want, 
so I'm going to switch to using functions to design the api and once 
the overall api design is complete, I'll re-evaluate if macros are a better fit.

In the next post, I'll add 'or' logic and allow for 
a [combination of and's and or's]({% post_url 2015-12-23-one-atom-database-combining-operators %})
