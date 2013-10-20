---
layout: post
title:  "When Objects Don't Need Behavior"
date:   2013-10-17 22:30:33
description: Some classes are better off as data structures with no behavior.
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

First of all, I would argue that `Person` is not a class, but a data structure. It's only purpose is to hold a set of data about a person

The problem with having properties on data structures is that they imply that there is behavior. Accessing data through properties is going through an
abstraction layer, and you never know when what's behind that abstraction can change. Even though we know the properties in `Person1` aren't doing anything,
client code using that class doesn't necessarily know.

I hear arguments that classes like `Person1` don't need to be unit tested because we are just setting properties and there is nothing to test, but
I would disagree.  I think the class above should be tested. On the other hand, I would argue that the functionally identical class `Person2` below does not need to be tested:

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

I think there's an important distinction between the two. `Person2`, allows us to directly manipulating the state of the Object. In the case of `Person2`, there really is nothing to test.

I'm definitely not saying don't ever use getters and setters. But I do think there is a subtle difference between an object and a data structure, and it might be worth considering.
