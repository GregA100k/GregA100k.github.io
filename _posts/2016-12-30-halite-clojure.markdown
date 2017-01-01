---
layout: post
title:  "Halite Conversion to Clojure"
date:   2016-12-30 12:00:00
categories: halite
tags: halite clojure programming
---
Writing Halite.io bots can be adicting.  The behavior of a bot can be tweaked in many ways and then watched using the 
Visualize option on the website.  It doesn't take long to have a number of different code paths depending on what 
is surrounding a particular square.  By the time my [original bot]({% post_url 2016-11-27-haliteintro %}) was working 
relativley well, the code had become pretty complex
and a new starter package was available for clojure, so I decided to rewrite the bot in Clojure with the hope that it
would be easier to build on.  This is the first time that I have taken existing Java code that I have written and have 
rewritten it in clojure.  It took longer than I expected.

Starter package differences
===========================

The biggest difference between the java starter package and the clojure starter package is that the java version calls
a move method each time it finds an owned square while the clojure package builds a list of owned squares and then
processes the whole list.  I had already made that same change to the java version so that was not an issue.  The
issue that I ran into was that I had a lot of nested logic.  I had an if statement checking for strength of zero so 
that square would not move.  I treated squares on the frontier different from the squares that were surrounded 
by squares that I owned.  I checked if it was possible for a square to overtake an adjacent square.  I checked if
one of the adjacent squares that I owned had enough strength to help to be able to overtake a square.  When I tried
to convert all this logic to clojure it looked horible.  

Method size vs Function size
============================

I don't know that directly translated logic really was any worse in Clojure, but it sure felt worse.  There doesn't
seem to be as much interest in smaller methods in Java as there is in other languages.  In [how much should global variables cost](http://www.benorenstein.com/blog/how-much-should-global-variables-cost) Ben Orenstein said there should
be a tax of $3.00 on methods over 5 lines long.  Looking for similar references for Java found mentions of
[hardly ever over 20 lines or 65 or more](http://softwareengineering.stackexchange.com/questions/133404/what-is-the-ideal-length-of-a-method-for-you)
and [less than 30](http://swreflections.blogspot.com/2012/12/rule-of-30-when-is-method-class-or.html).  For some reason,
the nested code in Java did not look as bad to me as it did in clojure.

By the second clojure version I had smaller methods and the code was getting a little easier to understand, but still had 
a lot of layers of checking if a rule applied and then applying that rule.

{% highlight clojure %}
  (if (= 0 (:strength site))
    :still
    (if (another-predicate site)
      (build-that-type-of-move site)
    ...
{% endhighlight %}

A bigger problem came with getting help from another square.  A helper square had to be removed from the list
of sites being processed which meant that a simple map or reduce no longer could be used to process the squares.  These 
two issues were solved together with [pie](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwjG9IOjg5rRAhWC7IMKHet-AOUQtwIIGjAA&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dlz1AnLn6f04&usg=AFQjCNGZ2WZRCzfsLqCR91uSLmgPhyM5Fg&sig2=BcLyFRfm_qDvuW2aiE-ocQ).


>
    Young Agent K: We need pie.
    Agent J: What?
    Young Agent K: My grand daddy always said, if you got a problem that you can't solve, helps to get out of your head. Pie, it's good.
>
https://en.wikiquote.org/wiki/Men_in_Black_3


![ski trail picture]({{ site.url }}/assets/IMG_0727M.JPG)

Who knew that cross country skiing is a valid substitute for pie. It may not be in most situations, but taking a ski break 
gave me a flash of insight that solved this code problem.

The complexity problem was solved by individual move functions that only return a move if the site meets the criteria.

{% highlight clojure %}
(defn zero-strength-move [site site-list game-map my-id]
  (if (= 0 (:strength site))
    [(vector (build-move site :still)) site-list]
    ))

(defn default-move [site site-list game-map my-id]
  [(vector (build-move site :still)) site-list]
  )
{% endhighlight %}

The key is the lack of else conditions.  Each individual move function is put into a list.  A some function
tries each function until it finds a move that applies.

The solution to the second problem of the helper sites can also be seen in these examples.  The calculated move
is being returned in a vector along with the rest of the sites yet to process.  For the squares that require help,
the move vector will have 2 entries and the list of remaining sites will have have the helper site removed.  A recursive
move function is called with the updated list of sites yet to be processed instead of mapping over a list.

{% highlight clojure %}
(defn move [moved site other-sites game-map my-id]
  (if (nil? site)
    moved
    (let [[moves remaining-others] (some #(% site other-sites game-map my-id) move-strategies)]
      (recur (concat moved moves) (first remaining-others) (rest remaining-others) game-map my-id))))
{% endhighlight %}

What's Next
===========

The current bot looks only at what is right around the square being moved, but the whole game map is available for
analysis. There might be a way to optimize the overall direction of attack by looking at the shape of the map.

