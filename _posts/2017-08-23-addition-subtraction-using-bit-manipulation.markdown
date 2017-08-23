---
layout: post
title:  "How to solve addition and subtraction using Bit manipulation"
date:   2017-08-23 15:03:26 -0700
categories: algorithm bit-manipulation
---
I was having problem understanding how to use bit manipulation to solve programming problems (in this case, they are addition and subtraction) until I had a converstation with my colleage. We had a really good and probably quite long converstation about how to come up with bit manipulation solution from scratch.

That's why I want to summarize everything here in order for me to understand them more clearly and hopefully help someones else who also are having similar problem.

**Addition**

So, let's start with Addition first.

Prob: How to implement `add(int a, int b)` function using bit manipulation?

Pretty clear, if my method receives 2 integers, it will return sum of those 2.

Now, how do you approach this problem? Hmm...

Let's try out some easy examples: (From now on, every nums I write is in binary)

Adding to binary numbers:

{% highlight HTML %}
{
	0 + 1 = 1 (easy)
	0 + 0 = 0 (easy)
	1 + 0 = 1 (easy)
	1 + 1 = 0 (this is interesting because you have additonal 1 value after addition, how do we address that?)
}
{% endhighlight %}

Let's not worry about the additional `1` for now. Based on above calculations we can have a truth table of sum operator
x | y | x + y | carryover
0 | 0 |   0   |    0
0 | 1 |   1	  |    0
1 | 0 |   1   |    0
1 | 1 |   0   |    1

How do you using bit manipulation to calculate the value of `+` operator? 
Looks similar? Yes it's `XOR` operator. So it's fair to say `x + y` equals `x^y + carryover`.

So the question is how do we calculate that carry over part? Let's take a closer look at it

`1 + 1` actually equals `10`. See the carry over `1` jumps to next slot which is multiplying with `2` (`2` is base system(binary), just like decimal is `10`: 5 + 7 = 1*10 + 2)

Ok, we are close, how to you represent "multiply with 2" using bit manipulation? Yes it's bit shifting (`<< 1` here we shift 1 bit to the left).

Let's look at above truth table again. If `x + y` is representation of `XOR` operator then what's carry over? It's `AND`.

Cool, finally we can complete our formula: `x + y` equals `x^y + (x&y << 1)`. And we keep doing it until there is no carry over (meaning `x&y << 1 == 0`) then at that time `x^y` is the result we are looking for.

**Subtraction (To be continued)**

Happy coding!

Hung Cao
