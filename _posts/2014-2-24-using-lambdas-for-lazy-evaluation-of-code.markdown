---
layout: post
title: Using Lambdas for Lazy Evaluation of Code
description: I'm such a procrastinator that I write code that procrastinates. At least, I do when I feel like it. But not right now.
date:   2014-2-24 22:00:00
comments: true
categories: programming
tags: lambdas delayed execution functional programming ruby
---
Recently I was introduced to the idea of using lambdas for delayed execution of code. In short, the idea is that instead of executing a piece of code immediately, you wrap it in a lambda function and wait to call the lambda until the computation is needed. Usually it's presented in the context of a really difficult problem like a decision tree with a lot of really complex computations that may or may not possibly need to be executed. But this past weekend I came across a pretty simple problem that lazy evaluation solved quite nicely and still shows how useful it can be.

I'm reading through *The Elements of Computing Systems* with the Seattle.rb study group at the moment, and in chapter 6 we have to build an assembler to translate assembly code into machine language. It's a pretty straight-forward program that goes through the assembly code line by line, looks up what the binary representation of the given instruction is, resolves memory addresses of variables and labels, and then outputs the translated machine code into a file.

One feature that is common in assembly languages is the ability to label a section of code for looping constructs. For example:

{% highlight ruby linenos %}
(MULT)
	@R1
	D=M

	@END
	D;JEQ

	@R1
	M=M-1

	@R0
	D=M

	@R2
	M=M+D

	@MULT
	0;JMP

(END)
	@END
	0;JMP
{% endhighlight %}

In this example there are two labels: ```MULT``` and ```END```. You can also see that there is a jump to 'END' on lines 5 and 6, essentially a goto statement. It's the assemblers job to see that ```END``` points to the memory address where the instruction on line 20 is stored, and substitute all the references to ```END``` with that address.

One complication in this example is that on line 6 we are jumping to ```END``` before the assembler has even reached line 20, where ```END``` is declared. So what does the assembler use for the address in the jump, when it doesn't know what address ```END``` points to yet? The book recommends processing the assembly file twice, the first time taking care of the labels and assigning them an address in memory, and the second time to substitute the references to those labels with the memory address when doing the transation to machine code. But I think there is a simpler solution that makes a pass over the code only once. 

The only difference with this solution is that instead of doing the work that converts the current line of assembly code to machine code, we just create a lambda function that *will* do the conversion at some point in the future. To be more precise, it will do the conversion *after* we have processed every line, and therefore all the symbols. Here's a pseudocode-ish example of what that might look like in Ruby:

{% highlight ruby %}
  def assemble(lines)
    instructions = []
    symbol_map = {}
    line_num = 0
    lines.each do |instruction|
      if is_symbol?(instruction) # if the instruction is a symbol, we need to store it in symbol_map.
        @symbol_map[instruction] = line_num)
      elsif is_symbol_reference? # otherwise we need to get the address of the instruction from symbol_map.
        instructions << ->() { symbol_map[instruction] } # notice we're not actually getting the address from symbol_map yet, just creating lambdas that *will* get it from symbol_map
      else
        instructions << ->() { assemble(instruction) }
      end
      line_num += 1
    end
    # now that each line and all the symbols have been processed, we can call the lambda functions.
    instructions.map { |lambda| lambda.() }
  end
{% endhighlight %}

The main point is that we're storing lambdas in the ```instructions``` array, and those lambdas will look into the ```symbol_map``` hash table to get the addresses. But we execute those lambdas only after each line has been processed, and therefore all the symbols have already been added into ```symbol_map``` at that point. Pretty cool if you ask me!
