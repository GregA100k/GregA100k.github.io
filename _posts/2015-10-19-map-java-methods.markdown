---
layout: post
title:  "Map with Java Methods"
date:   2015-10-19 21:30:00
categories: clojure java 
tags: clojure java interop
---
My most common use of clojre at work is to look at java objects.  I was looking at the available locales.  java.text.DateFormat.getAvailableLocales() returns a list of the locales but the return value doesn't help in seeing the locales so mapping over them sould show the locales.

{% highlight clojure %}
(map .toString (java.text.DateFormat/getAvailableLocales))

CompilerException java.lang.RuntimeException: Unable to resolve symbol: .toString in this context, compiling:(NO_SOURCE_PATH:1:1)
{% endhighlight %}

Putting the .toString inside an anonymous function does work.

{% highlight clojure %}
(map #(.toString %) (java.text.DateFormat/getAvailableLocales))
("" "ar_AE" "ar_JO" "ar_SY" ...
{% endhighlight %}

So clearly the java method is not a first class function. A little research in the clojure java interop page turned up the memfn macro which converts the java method into a function.  A note in the memfn doc says it's preferable to use the hash function syntax.

Looking at the code for memfn shows that all it is doing is wrapping the method in a function that is calling that method with the . special form.  I'd like to know why that isn't done by default, but need to do some research on how the . special form works

That will have to wait for another post.
