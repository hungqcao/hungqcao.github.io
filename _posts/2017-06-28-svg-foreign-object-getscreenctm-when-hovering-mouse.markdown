---
layout: post
title:  "SVG ForeignObject getScreenCTM calculates incorrectly after updating Chrome 59"
date:   2017-06-28 23:03:26 -0700
categories: svg d3 d3-tip getScreenCTM
---
Today, I ran over an issue with `SVG foreignobject` and `Chrome`. Basically, I needed to show a span inside a `D3` chart. Of course, there is only 1 way to display an `html` element inside a `svg` is using `svg:foreignObject`.

{% highlight HTML %}
{
	<foreignObject width="15" height="16" x="233.92277397260276" y="351">
		<div>
			<span class="bowtie-icon bowtie-symbol-crown" style="color: rgb(51, 153, 51);"></span>
		</div>
	</foreignObject>
}
{% endhighlight %}

After that, I need to display a tooltip using [d3-tip][d3-tip-url].
This is how d3-tip calculate the coordination of the tooltip:
`matrix = targetel.getScreenCTM`
> targetel is the current target of mouse over event, in this case is my `span`.

The problem immediately shows up here when `span` is not a `SVGElement` so it doesn't have `getScreenCTM` method. Easy fix! Just get its grand parent, in this case is `svg:foreignObject`:

{% highlight Javascript %}
{
	matrix = targetel.getScreenCTM ? targetel.getScreenCTM() : $(targetel).closest("foreignObject").get(0).getScreenCTM()
}
{% endhighlight %}

Everything worked fine before I updated Chrome to Version `59.0.3071.115 (Official Build) (64-bit)`. The `matrix.e` and `matrix.f` are not correct anymore. They are correct in Edge (Hmm I don't know how, maybe the other way around, will update when I have answer).

Anyway, the fix is very simple. Instead of using `x="233.92277397260276" y="351"`. Just use `transform="translate(338.20770547945204,351)"` on `svg:foreignObject`. (Again, I am not entirely sure why but it works for me).

Happy coding!

Hung Cao

[d3-tip-url]: https://github.com/caged/d3-tip