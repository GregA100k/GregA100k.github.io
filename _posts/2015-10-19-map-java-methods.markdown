---
layout: post
title:  "Map Over Java Methods"
date:   2015-10-19 21:30:00
categories: clojure interop
tags: clojure java interop
---
My most common use of clojre at work is to look at java objects.  I was looking at the available locales.  java.text.DateFormat.getAvailableLocales() returns a list of the locales but the array returned shows only that it is an array of Locales and doesn't help to learn which locales are available.  Mapping over the array sould show the locales.

{% highlight clojure %}
=> (.toString (second (java.text.DateFormat/getAvailableLocales)))
"ar_AE"

=>(map .toString (java.text.DateFormat/getAvailableLocales))

CompilerException java.lang.RuntimeException: Unable to resolve symbol: .toString in this context, compiling:(NO_SOURCE_PATH:1:1)
{% endhighlight %}

The toString method displays the locale, but trying to map it over the array does not work.

Putting the .toString inside an anonymous function does work.

{% highlight clojure %}
(map #(.toString %) (java.text.DateFormat/getAvailableLocales))
("" "ar_AE" "ar_JO" "ar_SY" ...
{% endhighlight %}

So clearly java methods are not first class functions. 

A little research in the clojure java interop page turned up the memfn macro which converts a java method into a function.  memfn source code shows that all it is doing is wrapping the method in a function which calls that method using the . special form.  I'd like to know why that isn't done by default, but will need to do more research on how the . special form works.

That will have to wait for another post.
