---
layout: post
title:  "Memoizing Tree Recursive Algorithms"
date:   2014-11-23 17:00:00
description: "Let's cross one off the bucket list: get a recursive implementation of Fibonacci to run in O(n)."
comments: true
categories: programming
tags: functional recursion javascript sicp
---
Computing the Fibonacci sequence is always a great way to introduce programmers to lots of important concepts: running time considerations, space considerations, recursion, dynamic programming, etc..

But our good friend `fib` has a bad rep because when implemented recursively it has an exponential run-time. In this post, I'm going to quickly describe a few of the common optimizations that address the run-time, compare them, and then present my favorite solution (hint: memoization) and describe how it works.

##Introducing Fib.
Here is the Fibonacci function in its simplest form, which computes the nth number of the Fibonacci sequence:
{% highlight js %}
function fib(n) {
  if (n == 0 || n == 1) {
    return n;
  }
  
  return fib(n - 1) + fib(n - 2);
}
{% endhighlight %}

If we take a look at what the evaluation of `fib(4)` looks like, we can see why this has an exponential run-time:

```
              fib(4)
             /      \
         fib(3)      fib(2)
         /   \        /   \
    fib(2)  fib(1)  fib(1) fib(0)
    /    \
fib(1)  fib(0)
```
This is what *tree recursion* looks like. In order to evaluate fib(4), we must evaluate fib(3) and fib(2). To evaluate fib(3), we need to evaluate fib(2) and fib(1), and so on.  There are a lot of redundant computations being performed. In the case of fib(4), we see that fib(2) has been evaluated twice, and fib(1) three times. When n gets large, this becomes exponentially worse and it quickly becomes almost impossible to run this algorithm for large numbers.

But ignoring that, there are some great things about this implementation as well. The way it is written is very natural and communicates what the Fibonacci sequence is in an intuitive way. In fact, it's exactly the mathematical definition of the function. Also, the space required is only the maximum depth of the tree.

##Some Optimizations
There are a couple ways to address the exponential run-time. We'll (very) quickly take a look at an iterative version of the algorithm, and also a solution where we store previously computed results so they don't have to be computed more than once.

Here is the iterative solution:

{% highlight js %}
function fib(n) {
  var x = 0;
  var y = 1;
  var z = 0;

  for(var i = 0; i < n; i++) {
    z = x + y;
    x = y;
    y = z;
  }

  return x;
}
{% endhighlight %}

The iterative solution is great because it has a linear run-time and constant space requirements. The downside is that this looks nothing like the mathematical definition of the Fibonacci sequence. The recursive solution is much easier to understand.

This next solution stores previously computed results in a lookup table. Before recursing, the algorithm first checks the lookup table to see if it has already been computed:

{% highlight js %}
var lookup = makeLookupTable();

function fib(n) {
  if (lookup[n]) {
    return lookup[n];
  }
  if (n == 0 || n == 1) {
    return n;
  }

  lookup[n] = fib(n - 1) + fib(n - 2);
  return lookup[n];
};
{% endhighlight %}

This one is easier to understand than the iterative solution. It also has a linear run-time, although the implementation of the lookup table will have an effect on both the run-time and the space required.

The problem with this one is that even though the mathematical definition of `fib` is there, it's hidden within all the logic around staring and retrieving previously computed values. We also now need to worry about that lookup table since it is in the global environment.

##Using Memoization

*Memoization* is storing previously calculated results to avoid repeated calculations. So we've already looked at memoization in action with the previous example. But if we use [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function), we can come up with a better solution.

One way to store a previously calculated value is with a memoization function. This is useful for expensive functions like `fib` which take a long time to compute.

{% highlight js %}
function memoize(func) {
  var result = null;

  return function memoizedFunc(n) {
    if (result) {
       return result;
    }

    result = func(n);
    return result;
  }
}
{% endhighlight %}

To describe how `memoize` works, I need to start using the term *environment*. Every time a function is called, the programming language's interpreter creates an environment that the function will be evaluated in. All the environment is, is a part of memory that has a key -> value map which maps variable names to their values. After the environment is created, all local variables and parameters for that function are placed in the environment so that the function has access to them. For example, when `memoize` is called, `result` and `func` will be placed in the environment. The other key point to understand is that `memoize` also return a function `memoizedFunc`, *which will have access to the environment that was just created* on each invocation. This means that `memoizedFunc` also has access to `result` and `func`.

Now we can memoize a function like this:

{% highlight js %}
// assign 'myMemoizedFunc' to 'memoizedFunc' by calling	'memoize'. Now on each call to myMemoizedFunc,
// it will have access to 'func' which is bound to 'functionThatTakesALongTimeToExecute', and 'result'.
var myMemoizedFunc = memoize(functionThatTakesALongTimeToExecute);

// 'functionThatTakesALongTimeToExecute' is evaluated on this first call to 'myMemoizedFunc', 
// and the computed value is assigned to 'result'.
var x = myMemoizedFunc(n);

// on the second call, the result has already been computed and can be returned immediately.
var y = myMemoizedFunc(n);
{% endhighlight %}

This is how we are going to attack the problem of memoizing Fibonacci. We'll have to try something slightly different however, because we need to memoize *all previously computed values* of the Fibonacci sequence, not just one value.

##Memoizing Fibonacci (finally)

Now that all of that is out of the way, we can start working on a memoized `fib`. For this to work, we need to define a lookup table where we can store each computed result in an environment that every call to fib will have access to. We can build off of our previous `memoize` function to create a new and improved one:

{% highlight js %}
function memoize(func) {
  var results = makeLookupTable();

  return function memoizedFunc(x) {
    var prevResult = results[x];
    
    if (prevResult) {
      return prevResult;
    }
    
    results[x] = func(x);
    return results[x];
  }
}
{% endhighlight %}

So far so good. But before we get to a working solution, I want to show you something that *will not* work. Understanding why this example *doesn't* work will help us understand the working solution.

{% highlight js %}
function fib(n) {
  if (n == 0 || n == 1) {
    return n;
  }
  
  return fib(n - 1) + fib(n - 2);
}

var memoizedFib = memoize(fib);

//this will NOT work as expected:
memoizedFib(100);
{% endhighlight %} 

Can you spot what the problem is? Let's remember what has to happen on each recursive call.  Every time we recurse, we need to update the lookup table to store the newly computed value. In order to do that, we need to call `memoizedFunc`.  We assigned `memoizedFib` to to `memoizedFunc` when we called `memoize(fib)`. Looking good so far. Next we call `memoizedFib(100)` which then calls `fib(100)`. This is where it breaks down. `fib` recursively calls itself, instead of recursively calling `memoizedFib`. If `memoizedFib` isn't being called, the table isn't getting updated.

##Introducing memoizedFib
Here's how we can get this to work:
{% highlight js %}
var memoizedFib = memoize(function(n) {
  if (n == 0 || n == 1) {
    return n;
  }

  return memoizedFib(n - 1) + memoizedFib(n - 2);
});
{% endhighlight %}

This has a subtle yet important difference. We are assigning memoizedFib to `memoizedFunc`, and then binding `func` to a new function which recursively calls `memoizedFib`. Now each recursive call to `memoizedFib` is executed in an environment that has access to the lookup table.

What I love about this solution is that it has the intuitive definition of the Fibonacci sequence, it doesn't contain the complexity of storing and retrieving previously computed values (since that responsibility has been handed off to `memoize`), and it has a linear run-time assuming the lookup table has a constant time lookup! 

##In Conclusion...
What someone might consider the best solution here is completely subjective and always depends on the circumstances.  There are no doubt many more optimized versions of `fib` out there.

But in the end we were able to do what we set out to do, which was memoize a tree recursive algorithm. I realize I hand waved over many important concepts such as the environment. If you find that interesting, the book *Structure and Interpretation of Computer Programs* shows how we can model function evaluation with *[the environment model of evaluation](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-21.html#%_sec_3.2)* which is a very useful tool to help you think through a problem such as this.