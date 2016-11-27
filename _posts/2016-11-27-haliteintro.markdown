---
layout: post
title:  "First Halite Experiences"
date:   2016-11-27 12:00:00
categories: halite
tags: halite java programming
---
The homepage of [Halite.io](https://halite.io/) 
says "Halite is an artificial intelligence programming challenge"  It is also a good way to 
suck up a long weekend.  

There are three official starter packages and a growing list of community packages.  These packages
have everything you need to start playing.  I chose the java package which has the MyBot.java 
class along with several helper classes all in the default package.  There is also a RandomBot.java
class that is a copy of MyBot.java.  It was interesting to me that the more I did, the more
software engineering practices came into play.

The first thing I wanted to be able to do was to improve on the default RandomBot.

{% highlight java %}
if(site.owner == myID) {
   Direction dir = Direction.randomDirection();
   moves.add(new Move(new Location(x, y), dir));
}
{% endhighlight %}
The default bot has the calculation of the move inside it so the first step is to extract 
that calculation into a separate class.

{% highlight java %}
BotMoveStrategy movestrategy = new SpecificMoveStrategy();
...
if(site.owner == myID) {
   Direction dir = movestrategy.calculateMove(site, currentLocation, gameMap);
   moves.add(new Move(new Location(x, y), dir));
}
{% endhighlight %}

The first strategy improvements follow the [Improving the Random Bot](https://halite.io/basics_improve_random.php) 
page.  Building on the idea that it's a bad idea to move a zero strength square, it also 
is a bad idea to attack a square that can't be taken.  It will reduce the strength of both
squares, but no new territory will be gained.  If the square stays still, there will be an
increase in strength so the initial strategy is to stay still until a square can be taken.

Once there are multiple squares, there is more to consider.  My initial strategy is to stay still 
as long as there is an unowned adjacent square.  Put another way, any square that is on the 
frontier will stay put until it's strength grows to the point were it can take over new 
territory.  This strategy holds until all the surrounding squares are owned and the square is
no longer on the frontier.  Strength in these internal squares, 
needs to be moved to create any value.  The questions of when to move and which direction 
need to be answered for that to happen.  As was already mentioned, when the strength is zero, 
it doesn't make sense to move.  Somewhere after strength has gone up from zero makes sense, 
but some sort of formula will be needed.  I don't have a good formula yet for when to start
moving.  Picking a direction to move is easier.  Since the strength will not be used until it
gets to the frontier, finding the shortest path to the frontier will add value the soonest.

The Frontier
============
The squares on the grid are not equal.  Each square has a strength, a production, and an
owner.  Owners are either me, an opponent, or the map (unowned pieces).  So far, I am not 
taking differences other than strenght into account when attacking.  Squares on the frontier 
attack any square that they can beat checking first to the North, then East, South, and West.
Since production values vary from square to square, taking control of higher production squares
is more valuable than controlling low production value squares.  Only the owned pieces gain 
strength so attacking a piece owned by an opponent is better 
than attacking a piece owned by the map since it would both increase my production and decrease
my opponents production.  

Observations on Game Play
=========================
Once a bot is submitted, the system starts playing it in games.  It seems that right after
a bot is submitted, it plays a lot of games in a short timespan.  After an initial flurry,
the games become less frequent.

It looks like most games are played against bots with similar rankings.  This screenshot
![Halite Game Feed Screenshot]({{ site.url }}/assets/20161127_haliteGameFeed.png)
shows one opponent in 3 out of 4 games and another opponent in 2 of the 4.  The screenshot
also shows that differences in the game boards can affect the outcome.  The two head to head
matchups have different winners.

My final observation is that rankings seem to go up over time.  I can't tell yet if that is 
because it takes time for a bot to climb up the rankings, or if it is because the bot found 
it's place in the list and some of the higher placed contestants have submitted new bots.

What's Next
===========
There are lots of places to improve my current bot.  Some of those changes will call for
more than just simple changes to the move strategy.  Before moving on, I'm putting the source
code into git so I can rollback experiments that don't work.

As far as the actual bot changes.  I've mentioned improving the simple North, East, South, West
strategy for picking where to attack.  I've also mentioned not having a good formula for when
to move internal strength.  A more complex change would be to find a way to calculate when
it's better to move strength off, or along, the frontier to attack a square sooner.
