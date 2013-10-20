---
layout: post
title:  "When Objects Don't Need Behavior"
date:   2013-10-17 22:30:33
description: Some classes are better off as data structures with no behavior.
comments: true
categories: programming
tags: objects structures propertioes oop encapsulation nitpick boring
---
I'll keep it simple for my first blog post.

I remember back when I was in college (okay, it really wasn't that long ago so it's not that impressive). 

Almost all the classes were taught in Java. Do you ever still have nightmares about your college days?  I do.
Nightmares with the professor at the front of the room shouting "Put getters and setters around all your member variables!"

Flash forward a few years and now I'm mostly working in C#. Not a whole lot has changed, except instead of getters and setters
C# developers use 'properties.' It's basically got the same purpose which is to provide some sort of interface to the internal
state of the object. It's become second nature to use properties instead of fields almost everywhere.

I see code like this quite a bit:

{% highlight csharp %}
public class Person1
{ 
    public string Name { get; set; }
    public string Occupation { get; set; }
    public int Age { get; set; }
    public int Weight { get; set; }
    ...
}
{% endhighlight %}

`Person1` is a class which follows that pattern.  Every member variable in that class is a property, and I admit it sure does look nice.

What about testing? would you bother writing unit tests for `Person1`? It seems like it would be pretty silly. I often see the argument that classes like `Person1` do not
need to be unit tested since we are just setting properties. And what's the point in testing that?

I would argue, in principle, that `Person1` should be tested. The reason why I say that is because by using properties we are implying that there is behavior. Accessing data through
 properties is introducing an abstraction around the internal state of that class, and you never know when the behavior behind that abstraction can change.
Even though we know the properties in `Person1` aren't doing anything, client code using that class doesn't know and shouldn't care.

On the other hand, I would argue that the functionally identical class `Person2` below does not need to be tested:

{% highlight csharp %}
public class Person2
{ 
    public string Name;
    public string Occupation;
    public int Age;
    public int Weight;
    ...
}
{% endhighlight %}

I think there's an important distinction between the two. `Person2`, allows us to directly manipulating the state of the Object and doesn't introduce any abstraction around that state.
In the case of `Person2`, there really is nothing to test.

So if you have some properties (or getters and setters) that you feel would be a waste of time to test, maybe they shouldn't be properties and you should keep them as public fields on that class.
