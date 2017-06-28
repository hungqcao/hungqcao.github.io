---
layout: post
title:  "A summary: how to use bit manipulation to solve problems easily and efficiently"
date:   2017-06-27 23:03:26 -0700
categories: algorithms bit-manipulation
---
**Wiki**
Bit manipulation is the act of algorithmically manipulating bits or other pieces of data shorter than a word. Computer programming tasks that require bit manipulation include low-level device control, error detection and correction algorithms, data compression, encryption algorithms, and optimization. For most other tasks, modern programming languages allow the programmer to work directly with abstractions instead of bits that represent those abstractions. Source code that does bit manipulation makes use of the bitwise operations: AND, OR, XOR, NOT, and bit shifts.

Bit manipulation, in some cases, can obviate or reduce the need to loop over a data structure and can give many-fold speed ups, as bit manipulations are processed in parallel, but the code can become more difficult to write and maintain.

**Details**
*Basics*

At the heart of bit manipulation are the bit-wise operators & (and), | (or), ~ (not) and ^ (exclusive-or, xor) and shift operators a << b and a >> b.
> There is no boolean operator counterpart to bitwise exclusive-or, but there is a simple explanation. The exclusive-or operation takes two inputs and returns a 1 if either one or the other of the inputs is a 1, but not if both > are. That is, if both inputs are 1 or both inputs are 0, it returns 0. Bitwise exclusive-or, with the operator of a caret, ^, performs the exclusive-or operation on each pair of bits. Exclusive-or is commonly abbreviated XOR.
* Set union A | B
* Set intersection A & B
* Set subtraction A & ~B
* Set negation ALL_BITS ^ A or ~A
* Set bit A |= 1 << bit
* Clear bit A &= ~(1 << bit)
* Test bit (A & 1 << bit) != 0
* Extract last bit A&-A or A&~(A-1) or x^(x&(x-1))
* Remove last bit A&(A-1)
* Get all 1-bits ~0

*Examples*

Count the number of ones in the binary representation of the given number

{% highlight Java %}
{
	int count_one(int n) {
		while(n) {
			n = n&(n-1);
			count++;
		}
		return count;
	}
}
{% endhighlight %}

Is power of four (actually map-checking, iterative and recursive methods can do the same)

{% highlight Java %}
{
	bool isPowerOfFour(int n) {
		return !(n&(n-1)) && (n&0x55555555);
		//check the 1-bit location;
	}
}
{% endhighlight %}

**^ tricks**

Use ^ to remove even exactly same numbers and save the odd, or save the distinct bits and remove the same.

Sum of Two Integers

Use ^ and & to add two integers

{% highlight Java %}
{
	int getSum(int a, int b) {
		return b==0? a:getSum(a^b, (a&b)<<1); //be careful about the terminating condition;
	}
}
{% endhighlight %}