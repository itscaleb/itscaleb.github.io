---
layout: post
title: OpenStruct All The Things!
description: You know what they say about ruby and developer happiness! Let's take a look at OpenStruct, a data structure in the standard library which allowed me to skip past a lot of hassle on a recent project.
date:   2013-10-20 23:57:30
comments: true
categories: programming
tags: openstruct ruby
---

Recently I've been writing a little prototype in my quest to learn about RESTful APIs, and in particular, what advantages hypermedia brings to your API. This post isn't about REST, but it's about a little gem I discovered while working on my prototype. Let's take a quick look at a data structure in the ruby standard library called OpenStruct, which I found to be *very* helpful while prototyping.

I decided to build a REST API for a simple book library management system. I'm building it in ruby using Sinatra and MongoDB, a NoSQL document database. 

For a little bit of context, I work at a .NET shop and I'm fairly new to ruby. So in what is probably a typical approach for someone used to statically typed languages, I started to build class data structures to deserialize the MongoDB documents into. When using the MongoDB ruby driver, documents are stored and loaded from the database as ruby hashes. This might give you an idea of what my approach was like for each type of document that was stored:

{% highlight ruby %}
# get a book document out of the database
book_hash = collection("Books").find_one("_id" => id)

# create an instance of a book data structure from that document
book = Book.from_hash(book_hash)
{% endhighlight %}

`Book` has a `from_hash` method that takes a hash, and returns a new instance of the `Book` class with it's instance variables set from the values of that hash. This approach wasn't horrible at first, but it quickly became a chore. I found myself frequently changing the structure of the documents I was storing in MongoDB, especially as I was ironing out the details of how I wanted to model my data. And with the increasing amount of classes similar to `Book`, I was spending way too much time going back and forth between changing the way I was storing the data in MongoDB and fixing the `to_hash` methods in all of those classes to reflect those changes. Luckily I found a solution to help ease my pain called OpenStruct.

## OpenStruct to the rescue!

`OpenStruct` allows you to set arbitrary values to arbitrary attributes.  Here's an example ripped off from the documentation:

{% highlight ruby %}
person = OpenStruct.new
person.name = "John Smith"
person.age = 70

puts person.name     # -> "John Smith"
puts person.age      # -> 70
{% endhighlight %}

What's even cooler is that OpenStruct has a constructor that takes a hash and builds an object off of it! So my earlier code has now turned into this:

{% highlight ruby %}
# get a book document out of the database
book_hash = collection("Books").find_one("_id" => id)

# create an OpenStruct
book = OpenStruct.new book_hash

# access data normally from the OpenStruct
puts book.author	# -> "I.P. Freely"

# I can even extend it with modules
book.extend(BookRepresenter)

# and I can call the 'to_h' method to put it back into a hash to store in MongoDB
collection("Books").insert(book.to_h)

# all of this without even defining a "Book" class.
{% endhighlight %}

After going with this approach, I was able to remove about 6 or so un-needed classes from the code base.  Plus it is *very* helpful when dealing with hashes directly out of a document database like MongoDB. I love how whenever I modify how the data looks in MongoDB, the OpenStruct object instantly reflects those changes.

I haven't seen a class similar to this in .NET, although `ExpandoObject` comes close. You can't constuct an ExpandoObject from an IDictionary, but writing a `FromDictionary` extension method for ExpandoObject is probably quite doable. 

There could potentially be a few downsides to using OpenStruct in this manner. There's probably some performance considerations for example. But I think for simple data structures (and definitely for quickly prototyping) using OpenStruct can be a great choice.
