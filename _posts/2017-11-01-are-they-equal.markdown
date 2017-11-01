---
layout: post
title:  "Are They Equal?"
date:   2017-11-01 05:00:00
categories: clojure
tags: clojure programming
---

Beginner Night Topics
=====================
At the September Clojure MN meetup, there was a brief discussion of the = function.
The usual discussion about comparison etc.

defn came up later in the talk with a common example 

{% highlight clojure %}
(defn myadd1 [a b]
 (+ a b))
{% endhighlight %}

and the explanation that defn is a macro that gives syntactic sugar for 

{% highlight clojure %}
(def myadd2 (fn [a b] 
  (+ a b)))
{% endhighlight %}

and those are two ways to accomplish the same task.  

But are they equal?
===================

Someone then asked if the two functions created from different methods are equal.

{% highlight clojure %}
user=> (defn myadd1 [a b] (+ a b))
#'user/myadd1
user=> (def myadd2 (fn [a b] (+ a b)))
#'user/myadd2
user=> (= myadd1 myadd2)
false
{% endhighlight %}


My first thought following this is that = is just comparing the object IDs like the java ==.  If that is true then
then comparing two vectors with = should also return false.

{% highlight clojure %}
user=> (= [1 2 3] [1 2 3])
true
user=> (= [1 2 3] [1 3 2])
false
user=> (= '(1 2 3) [1 2 3])
true
user=> (= '(1 2 3) '(1 3 2))
false
{% endhighlight %}

Clearly vectors and lists are working differently.  The two functions are different objects and which are not equal, but vectors
and lists are looking inside the object and comparing the contents.  This is certainly worth a deeper look.

How does = work?
================

The first step to look at = is to look at the source code.  This is easy to do in the REPL.

{% highlight clojure %}
user=> (source =)
(defn =
  "Equality. Returns true if x equals y, false if not. Same as
  Java x.equals(y) except it also works for nil, and compares
  numbers and collections in a type-independent manner.  Clojure's immutable data
  structures define equals() (and thus =) as a value, not an identity,
  comparison."
  {:inline (fn [x y] `(. clojure.lang.Util equiv ~x ~y))
   :inline-arities #{2}
   :added "1.0"}
  ([x] true)
  ([x y] (clojure.lang.Util/equiv x y))
  ([x y & more]
   (if (clojure.lang.Util/equiv x y)
     (if (next more)
       (recur y (first more) (next more))
       (clojure.lang.Util/equiv y (first more)))
     false)))
nil
{% endhighlight %}

There are a couple of interesting things to notice.  The first thing that stands out is that = can be called with 
only one object.  While that may not be a task that would ever be done by design, it does seem like it could be useful 
in a dynamic situation to avoid extra existence checks.  The second thing to notice is that the bulk of the work 
here is being done in java.  Looking at the Java source is a little more work than running a command in the repl, but it
is available at [clojure.lang.util](https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/Util.java)

{% highlight java %}
static public boolean equiv(Object k1, Object k2){
	if(k1 == k2)
		return true;
	if(k1 != null)
		{
		if(k1 instanceof Number && k2 instanceof Number)
			return Numbers.equal((Number)k1, (Number)k2);
		else if(k1 instanceof IPersistentCollection || k2 instanceof IPersistentCollection)
			return pcequiv(k1,k2);
		return k1.equals(k2);
		}
	return false;
}

static public boolean pcequiv(Object k1, Object k2){
	if(k1 instanceof IPersistentCollection)
		return ((IPersistentCollection)k1).equiv(k2);
	return ((IPersistentCollection)k2).equiv(k1);
}
{% endhighlight %}

Here we can see that [Numbers](https://docs.oracle.com/javase/8/docs/api/java/lang/Number.html) and 
persistent collections are in fact handled separately from other types of objects.  All other types of objects, no matter if 
they are java objects like Strings or Clojure created objects like user defined functions, drop down to calling the equals 
method of that object.

For persistent collections, pcequiv calls equiv method of IPersistentCollection which is fully into the OO world of
Java.  This might be easier to work through with a specific example like Vectors.  Here is a picture of the 
class hierarchy for Vectors.


![icon picture]({{ site.url }}/assets/clojurePersistentVectorClass.png)

[PersistentVector](https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/PersistentVector.java) inherits 
behavior from a number of places 
including [IPersistentCollection](https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/IPersistentCollection.java) 
which is an interface, so it only defines the equiv method, the implementation for vectors is found in 
the [APersistentVector](https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/APersistentVector.java) abstract 
class.

{% highlight java %}
public boolean equiv(Object obj){
    if(obj == this)
        return true;
	return doEquiv(this, obj);
}
{% endhighlight %}

The real work of comparing the vectors is in doEquiv which walks through the elements and recursively returns to the 
Util.equiv method for each element in the vector.

Back to Functions
=================

So how does = work for functions.  Since functions do not match any of the if conditions of Util equiv method, the comparison
is done in the line
		return k1.equals(k2);
which calls equals on the java class that is created for the user defined function.

I could think of two choices for finding out how the equals function is defined for a user defined function.  One way would be
to dive into the code for [compile](https://clojuredocs.org/clojure.core/compile) and learn how that works.  The other 
would be to define a function and compile that into java code and then decompile the results.  I chose the second path 
as it seemed like it would be much easier.

Here is a small piece of clojure code creating two functions
 
{% highlight clojure %}
(ns comparefunc.core
  (:gen-class))

(defn x1
  [x]
  (+ x 1))

(def x2 (fn [x] (+ x 1)))

(defn compare-functions []
  (println "compare functions directly " (= x1 x2))
  (let [x3 x1]
    (println "compare same function " (= x1 x3)))
)

{% endhighlight %}


Calling compare-functions behaves as expected.  x1 and x2 do the same thing, but they are not equal. 

{% highlight shell %}
C:\Users\ga1\dev\apps\comparefunc>java -cp c:\lang\clojure-1.8.0\clojure-1.8.0.jar;.;.\src;.\classes clojure.main
Clojure 1.8.0
user=> (require 'comparefunc.core)
nil
user=> (comparefunc.core/x1 2)
3
user=> (comparefunc.core/x2 2)
3
user=> (comparefunc.core/compare-functions)
compare functions directly  false
compare same function  true
nil
{% endhighlight %}


I started the repl without the help leiningen to show that the classes folder is part of the classpath.  
The [compilation](https://clojure.org/reference/compilation) page mentions that the compile-path must be set
and [defaults to classes](https://stackoverflow.com/questions/26385333/gen-class-does-not-generate-a-class-file)

{% highlight shell %}
user=> (compile 'comparefunc.core)
comparefunc.core
user=>
{% endhighlight %}

Each function gets compiled into its own class file which I decompiled.  For example, the x1 function 
compiled into core$x1.class 


{% highlight java %}
package comparefunc;

import clojure.lang.AFunction;
import clojure.lang.Numbers;

public final class core$x1 extends AFunction {
	public static Object invokeStatic(Object x) {
		Object arg9999 = x;
		x = null;
		return Numbers.add(arg9999, 1L);
	}

	public Object invoke(Object arg0) {
		Object arg9999 = arg0;
		arg0 = null;
		return invokeStatic(arg9999);
	}
}
{% endhighlight %}

The generated class does not define an equals method so it will use the inherited equals function from Object which checks
only if the two referenced objects are the same object which the compare-functions call has already shown to be true.

In the end, comparing functions is not very exciting.  There is no magic, and it was as expected just using the Object.equals method.
