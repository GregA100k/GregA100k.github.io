---
layout: post
title:  "My Intro to om"
date:   2015-11-03 22:00:00
categories: clojure 
tags: clojure
---
I wrote a web site to learn om.  I volunteered to score a six hour running
race, the 
[NightOwl Shuffle](http://www.nightowlshuffle.org/)
so the site was to track the runners.  The race has a long and a short loop, 
so each time a runner came through, I would enter the runner's number and which
loop as the runner went past.   The site would add up the mileage
and adjust the standings.  

The site used om and was developed using Figwheel.  I used the 
[Quick Start](https://github.com/bhauman/lein-figwheel/wiki/Quick-Start) 
for Figwheel and followed the
[Basic Totorial](https://github.com/omcljs/om/wiki/Basic-Tutorial) and 
[Intermediate Tutorial](https://github.com/omcljs/om/wiki/Intermediate-Tutorial)
for om.

**Figwheel is the coolest!!**

I worked with the browser open in the background and an editor open
in front.  Save a file. The page updates.  Change branches in git. 
 Page updates.  Spend a day waiting fifteen minutes for a j2ee instance 
to fire up after every change and that is the greatest thing in the world.
For a lot of changes, I didn't even have to swap over to the browser to see
the effects of a code change.  It just showed up in the browser 
in the background.

**Back on topic**

For my race scoring website, I had runners, courses (the long and short lap 
courses), and then when the race starts, laps which are a combination of a 
runner and a course.  A many to many relationship.  
Laps would be an intermediate table with foreign keys to runners and courses.

Of course within an Om application, there is no database.  According to the 
[Basic Tutorial:](https://github.com/omcljs/om/wiki/Basic-Tutorial) 

>In Om the application state is held in an atom, 
>the one reference type built into ClojureScript

Within that atom, I had a keyword key of :runners, a keyword key of :courses
and a keyword key of :laps  Each of those keys has a value that is a vector
of maps.

To create the runners page, I needed the list of runners.  Each of the first
pages built needed the list under it's key.

{% highlight clojure %}
(:runners @app-state)
{% endhighlight %}

If I wanted one particular runner I had to filter those results

{% highlight clojure %}
(filter #(= (:race-number %) *desired-value*) (:runners @app-state))
{% endhighlight %}

This type of code is scattered all over the big ball of mud that became
my race scoring site.  What I'd like is to be able to treat the app-state
atom like a database and query it in a familiar way.

That will be the topic for the next post
