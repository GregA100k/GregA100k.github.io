---
layout: post
title:  "Clojure Help Chrome Extension"
date:   2017-03-12 12:00:00
categories: halite
tags: clojure clojurescript chrome extension programming
---
Overview
========

Two related ideas came together one day this winter. I listened to a 
[Javascript Jabber](https://devchat.tv/js-jabber/233-jsj-google-chrome-extensions-with-john-sonmez)
episode about Chrome extensions and received an email saying that [Clojre.mn](https://www.meetup.com/clojuremn/) was looking
for presenters.  It seemed like a good match and this is my introduction to Chrome Extensions.

So what is a Chrome Extensions?  According to [chrome developer documentation](https://developer.chrome.com/extensions)
>Extensions are small software programs that can modify and enhance the functionality of the Chrome browser. You write them using web technologies such as HTML, JavaScript, and CSS.  It all starts with a manifest file, a json file describing the pieces of the 
extension.

A manifiest.json file specifies the actions that the extension will perform.  Browser actions apply to all web pages 
as opposed to page actions which only apply to specific pages.  I want to have an extension that will let me highlight a 
clojure function on any web page and let me look up the documentation on the
[Clojure CheatSheet](https://clojure.org/api/cheatsheet) so browser action it is.  

{% highlight json %}
"browser_action": {
   "default_title": "Clojure Help",
   "default_icon": "crappyHandDrawnLambda.png",
   "default_popup": "popup.html" 
 },
{% endhighlight %}

The json in the manifest describes the three pieces of the browser action.  The icon displays next to the address 
bar.  ![icon picture]({{ site.url }}/assets/chromeLambdaIcon.png)

The title is optional and shows up as a tool tip when you hover over the icon.  When you click the icon, the popup html page is 
activated.  

The popup page is where the help information will be shown, but there is still a missing piece to find out the 
topic of that help page.  [Content Scripts](https://developer.chrome.com/extensions/content_scripts) provide a mechanism
to run javascript inside the context of a web page.  The manifest.json file is, once again, the way to tell chrome to run 
the specified javascript.


{% highlight json %}
 "content_scripts": [
   {
     "matches": ["<all_urls>"],
     "js": ["selection.js"],
     "run_at": "document_start",
     "all_frames": true
   }
 ]
{% endhighlight %}

I was certainly not the first person who wanted to be able to read highlighted text in a chrome extension and since
I was not going to be doing anything special to find it, I looked to [Stack Overflow](http://stackoverflow.com/questions/19164474/chrome-extension-get-selected-text)
to find an already working solution. 

The popup html page
===================

I started with a very simple html page. All the html page needs to do is load the javascript, and provide a spot
where the content can be placed.


{% highlight html %}
<!DOCTYPE html> 
<html>
<head>
    <script type="text/javascript" src="out/goog/base.js"></script>
    <script type="text/javascript" src="out/popup.js"></script>
    <script type="text/javascript" src="test.js"></script>
<style>
  body { width: 600px; height: 400px; margin: 0; padding: 0;}
</style>

</head>
<body>
  <div id="spot" />
</body>
</html>
{% endhighlight %}

Clojurescript Quick Start
=========================
The javascript is included, so now, there needs to be some javascript to take the highlighted text and display.  That 
javascript will be the compiled output of clojurescript code.  The [Clojurescript Quick Start](https://github.com/clojure/clojurescript/wiki/Quick-Start)
describes how to make that happen.  I was thrilled to see that this page actually explains how to do the compile
and does not say 'just use leiningen'.

instead, they give a simple build.clj source file

{% highlight clojure %}
(require 'cljs.build.api)

(cljs.build.api/build "src" {:output-to "out/popup.js"})
{% endhighlight %}


 and a java command to run the compile. 
{% highlight java %}
 java -cp "cljs.jar;src" clojure.main build.clj
{% endhighlight %}


Popup
=====
{% highlight clojure %}
(defn pasteSelection []
  (let [f1 (fn [tab]  (.sendMessage js/chrome.tabs 
                                   (aget tab "id")
                                   #js {:method "getSelection"} 
                                   handle-highlighted-text))
       ]
  (current-tab f1)))

(set! (.-onload js/window) pasteSelection)
{% endhighlight %}


Working up from the bottom, the popup window will run the pasteSelection function as soon as it 
is loaded.  This requires sending a message to the javascript function that was loaded on the page asking
for the highlighted text.  The final parameter of the message is a callback function that will process
whatever is highlighted on the page.

Handling the highlighted text involves checking to see if the text matches a link from the clojure cheatsheet and if it does, redirect the popup to the page referenced in that link

Walking clj-tagsoup
===================

The original goal of the extension was to dynamically parse the Clojure Cheat Sheet to find all the links, but the hurdles put
in place to prevent
[cross site scripting concerns](https://en.wikipedia.org/wiki/Cross-site_scripting) threats make that difficult so I decided
to parse the cheatsheet separate
from the Chrome extension and manually move the output of that parse into the extension code.

Parsing the Cheat Sheet html into a clojure data structure could not be easier than with 
[clj-tagsoup](https://github.com/nathell/clj-tagsoup).  Only one command.  Finding the links in that data structure took a
little more code using the [clojure.walk](https://clojure.github.io/clojure/clojure.walk-api.html) library. 

{% highlight clojure %}
(defn -main [& args]
  (def cheatsheet (ts/parse "https://clojure.org/api/cheatsheet"))
  (walk/postwalk my-fn cheatsheet)
  (doall (map (fn [[k v]] (println (str "\"" k "\"  \""  v "\""))) @linkmap))
)
{% endhighlight %}



The API documentation says that clojure.walk 
>makes it fairly easy to write recursive search-and-replace functions, as shown in the examples.

Based on the descriptions of the API, I was pretty sure that I wanted to use postwalk to process the urls.  I need to find a 
complete url structure and, once it has been found, I want to extract the text on the link, and the URL.  The postwalk 
function "performs a depth first post-order traversal" which means that the entire link structure will be passed to my 
function.  Based on the advice of the API documentation, I looked to the examples to make sure I had the correct function. In the 
case of postwalk, even the example took some unwinding.

{% highlight clojure %}
(use 'clojure.walk)
(let [counter (atom -1)]
   (postwalk (fn [x]
                [(swap! counter inc) x])
              {:a 1 :b 2}))

=> [6 {2 [[0 :a]  [1 1]], 5 [[3 :b]  [4 2]]}] 
{% endhighlight %}

At first, this simple looking example is a bit hard to follow with single digit numbers representing two different things.  Each 
time the anonymous function is called, it returns a vector with the increased counter as the first element and the element that
it is processing as second.  In this case, the tree that is being walked is a map with two key-value pairs.  Since
the postwalk is depth first, it sees that the map has sub elements and so it recursively goes into the first key-value 
pair.  That first key-value pair also has sub elements so it recursively calls the the key :a.  Since :a does not
have any children, it calls the anonymous function which returns a vector containing the incremented counter and :a
> [0 :a]
It does the same for the second key-value pair, then starts working it's way outward.  Counter 1 is assigned to 
the value of the first pair, counter 2 is assigned to the first key-value pair, counters 3, 4, and 5 are similarly
applied to the second key-value pair, and finally counter 6 is assigned to the entire map.

The function passed to postwalk needs to do two things.  First, it needs to identify the links, and second, it needs to 
extract out the text and url.  All the links on the page are in the form of a vector where the first value is an anchor
tag, :a, the second element is a map containing the title, url, and other properties, and the third element of the vector is the 
text displayed on the link.  

{% highlight clojure %}
[ :a { :shape rect,
    :href https://clojuredocs.org/clojure_core/clojure.core/max, 
    :title clojure.core/max ([x] [x y] [x y & more]) Returns the greatest of the nums.}
  max
 ]
{% endhighlight %}

Selecting vectors that :a for the first element applied to more links than were wanted, so I added another check 
to verify that a :title element was present.  If an element started with :a, and had an :href and a :title in the map that was 
the second element of the vector, then I add the text from the link and that url to a map


{% highlight clojure %}
(def linkmap (atom {}))

(defn my-fn [l]
  (let [firstl (if (vector? l) (first l))
        p1 (if (and (= :a firstl)
                    (:href (second l))
                    (:title (second l))
                )
                (swap! linkmap assoc (last l) (:href (second l)))
                )
        ]
    l))
{% endhighlight %}

This is a bit of a hack, but I printed that map, and brought that into the clojurescript code to lookup 
urls. 


Putting it all together
=======================
With the clojurescript compiled, the html loading the javascript, and the manifest telling chrome what the pieces are, 
the next thing is to load this all into chrome.  This is done on the Extensions page under More Tools.  On the 
extensions page, check the box for Developer mode near the top.  This allows loading extensions from a local 
directory.  Press the "Load unpacked extension..." button and navigate to the folder containing the extension.


Learnings and Demonstrations
============================
Live demos will always teach a lesson.  I tried to present this work at the  [Clojre.mn](https://www.meetup.com/clojuremn/) meeting 
where I learned one more important fact.  There is a piece of code
which finds the current tab in chrome.  It is used by the extension to know where to ask for highlighted text.  This code did 
not work while inside a Google Hangout while sharing the desktop.  I think the reason for this is that Google puts a small window
on top of everything when the desktop is shared.  I believe this is where the request for highlighted code was being sent, and
the highlight was never found.

Links
=====
* [cljhelp](https://github.com/GregA100k/cljhelp) is the project for putting together the Chrome Extension
* [csparse](https://github.com/GregA100k/csparse) is a helper project to parse, extract, and print all the urls from
the clojure cheatsheet.
