---
layout: post
title:  "Keep it simple, don't add behavior to that object."
date:   2013-10-17 22:30:33
description: Often times objects don't need behavior. Don't add it if you don't need it.
tags: objects structures oop encapsulation nitpick boring
---
I'll keep it simple for my first blog post.

I remember back when I was in college (okay, it really wasn't that long ago so it's not that impressive). 

Almost all the classes were taught in Java. Do you ever still have nightmares about your college days?  I do.
Nightmares with the professor at the front of the room shouting "Put getters and setters around all your member variables!"

Flash forward a few years and now I'm mostly working in C#. Not a whole lot has changed, except instead of getters and setters
C# developers use 'properties.' It's basically got the same purpose which is to provide some sort of interface to the internal
state of the object.

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

First of all, I would argue that `Person` is not a class, but a structure. So I might have lied a little bit in the title of this
blog post. It's perfectly fine for an object to have behavior, but maybe it's not so easy to determine when an object is really an object, or if it's
just a structure in disguise.

Anyways, the problem with having getters, setters or properties on structures, is that they imply that there is behavior. They're encapsulating the
internal state of the object. And if getters and setters are providing behavior rather than state, than that behavior can change while
the interface of that object remains the same.

I hear arguments that classes like the one above don't need to be unit tested because we are just setting properties and there is nothing to test.
I would disagree.  I think the class above should be tested. On the other hand, I would argue the class below does not need to be tested:

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

I think there's an important distinction between the two. With `Person1`, we are using an interface that gives us access to the internal state of the object.
With `Person2`, we are directly manipulating the state of the Object. In the case of `Person2`, there really is nothing to test.

I'm definitely not saying don't ever use getters and setters. But I do think there is a subtle difference between
an object and a structure, and it might be worth considering.
