---
layout: post
title: Events vs. Callbacks
---
This afternoon at work a co-worker was asking about an older, custom, well established, and prevalent jQuery plug-in present in our code base. 
He was attempting to identify how to override the click event of the button in the object spawned by the plug-in. 

This plug-in at first inception contained two callbacks relevant for his needs, a `updateHandler` and a `selectionHandler` - as the plug-in has aged, a third method has been added `clickHandler`.

All three methods perform very similar function, with some subtle differences.

I informed him he probably wanted the `clickHandler` and he asked for an example to look at.

"Ehh, just search the scripts folder for `clickHandler` - I tend to avoid such awful names in functions, so it should only appear in usage of this plug-in"

## The Cause
At this point, our lead - and originator of the plug-in - chimed in, "What do you mean awful!?"

Flustered, I rebuffed... well its OK, I added the method, its OK if I call it awful! I was just following the convention of the plug-in.

I was just following the existing convention... 

I have been mulling this conversation over for the better part of the evening and think I know why I decided to deem the `clickHandler` callback _awful_. It wasn't because it was a bad method, or a bad method name,
or that it did something awful, or that it shouldn't exist... I think simply put, my issue is that it is a *callback*

## The Thought
In the Javascript programming language, dealing with asynchronous code tends to fall into one of two camps:
*Callbacks* or *Events*

The distinction between these two ideas is pretty simple: a callback is some argument provided to a function be executed at the appropriate time with the appropriate arguments, whereas, an event is some function 
bound to an event rail and notified at a designated time with appropriate arguments.

For example, a callback takes some form like:

```
	var some_long_operation = function(callback) {
		//some long operation
		$.ajax({
			success: function(data) {
				callback(data.property);
			}
		});
	}

	some_long_operation(function(someArg) {
		console.log(someArg)
	});
```

And an event rail looks more akin to:

```
	var target = $(this); //some object to house the rail, could be a dom object, most likely a javascript object
	
	target.on("operation.finished", function(e, someArg) {
		console.log(someArg);
	});
	
	var some_long_operation = function() {
		//some long operation
		$.ajax({
			success: function(data) {
				target.trigger("operation.finished", [data.property]);
			}
		});
	}
	
	some_long_operation();
```

Now, on the surface, I don't really believe either of these is directly superior in all circumstances. 

*Callbacks* are fantastic for methods that likely change with every usage, and can have their entire behaviour categorized in a singular method. A good example would be adding a new dom element to the page after successfully fetching
a data set to push into your template. The behaviour is likely small, and different with every implementation - it is also usually pretty singular in scope, meaning it can encompass all of the desired code in a single function.

*Events* really shine when you want do a potentially infinite number of things with the data. For example, I may want to fetch new notifications a user has received every 15 seconds. When I finish loading the notifications, 
I want to repaint the notification panel, add a message to the messages panel, increment an un-read notification indicator, fire off a sound byte indicating a new notification, etc. Utilizing a callback for this would turn into a nightmare,
as every feature you add to the page that wants to know about new notifications would need to be added to the "god" callback handling 100% of a new notification receipt.

In the notification example, a callback would be very cumbersome, as the single script responsible for fetching the notifications would also need to know about potentially infinite UI elements, sound files, etc.
If we utilize an event and notify the body (or some target, not important) of new notifications, literally separate script _files_ can take on their unique roles in handling the new notifications.

## Why Did I Care?
In the situation today, I realized I thought the `clickHandler` was awful because I have come to realize I chose the wrong thing for this plug-in. I followed the existing pattern in the plug-in, and added another callback.

Looking at the plug-in now, and knowing what I do about how I want to interact with the plug-in - and how my co-worker wanted to interface with it today - Adding an event to the plug-in would have been a much less disruptive edit.

Think about it, when I added the `clickHandler` I boldly made the decision that the plug-in could have exactly one point of control at the moment the button was clicked. 

The goal of utilizing the `clickHandler` today was to update a handful of UI elements, and fire of a POST command via ajax. _Those two operations do not belong together._ But by utilizing the callback in this case, I had forced that upon 
my implementer. Sure, they could separate out the behaviours into nice functions and call out to those, that's _marginally_ better, but in my eyes updating DOM and firing an ajax command are exclusive things that shouldn't happen 
in the same function (usually!).

## Closing Thoughts
I'm not advocating that anyone reading this decide "From now on, events only" - that would be crazy.

I am however, wanting to pass on a situation that made me realize _why_ I thought an older piece of code was sub-optimal. I regret designing it the way I have, because it has caused some very awkward monolithic functions that do _waaay_ to much. 

I followed the convention of the plug-in, and have paid a penalty often. The convention is hardly bad or incorrect, I just blindly followed it and it took me too long to see my error, and now its just the _awful_ `clickHandler`.
