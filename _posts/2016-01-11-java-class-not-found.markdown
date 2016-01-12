---
layout: post
title:  "Jave Class Not So Much Found"
date:   2016-01-11 22:00:00
categories: java
tags: java
---
I work on a java monolith application that has it's own version of cron jobs
and during our last implementation, I noticed that one of these jobs 
had started dumping "java.lang.NoClassDefFoundError" messages each time it ran.

First assumption was that there had been a problem with the deployment
and something was missing.  After checking jar files and seeing that the 
class had been deployed, I went back to the beginning of the server startup
and found that the first time the cron job ran, it threw a different error.


Here is a simplified example of the problem.  The first class is just
to throw a runtime exception in the static initializer section of the 
second class.

{% highlight java %}
public class Crasher {

	public void kaboom() throws RuntimeException {
		System.out.println("KaBoom!");
		throw new RuntimeException("kaboom");
	}
}

public class WithStaticInitializer {
	static {
		System.out.println ("Inside the static init block");
		Crasher c = new Crasher();
		c.kaboom();
	}

	@Override
	public String toString() {
		return "WithStaticInitializer";
	}
}
{% endhighlight %}

The final piece is a class to run the test.  The first time that an instance
of the class WithStaticInitializer is instantiated, it throws a 
java.lang.ExceptionInInitializerError exception.  Inside the same JVM, if 
another attempt is made to instantiate the class, a different exception is 
thrown, java.lang.NoClassDefFoundError.  The message for this exception
says that it could not instantiate the class.  The println inside the 
static block shows that the jvm doesn't like to waste time and only tries
to intialize the class the first time.  Subsequent calls throw the exception 
without even trying to load the class.

Here is the runner program.

{% highlight java %}
public class Runner {

	public static void main(String[] args) {
		try {
			WithStaticInitializer wsi = new WithStaticInitializer();
			
			System.out.println("Won't get here " + wsi.toString());
		} catch (Throwable t) {
			System.out.println("First message "
					+ t.getClass().getName() + " :: "
					+ t.getMessage());
		}
		
		try {
			WithStaticInitializer wsi2 = new WithStaticInitializer();
			
			System.out.println("Won't get here " + wsi2.toString());
		} catch (Throwable t) {
			System.out.println("Second message "
					+ t.getClass().getName() + " :: "
					+ t.getMessage());
		}
	}
}
{% endhighlight %}

And here is the output.

{% highlight java %}
Inside the static init block
KaBoom!
First message java.lang.ExceptionInInitializerError :: null
Second message java.lang.NoClassDefFoundError :: Could not initialize class com.ga100k.statinit.WithStaticInitializer
{% endhighlight %}

