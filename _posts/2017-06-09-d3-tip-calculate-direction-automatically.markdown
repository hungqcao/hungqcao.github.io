---
layout: post
title:  "Making D3 tip to calculate its direction automatically"
date:   2017-06-09 23:03:26 -0700
categories: vso-dashboard, d3, d3-tip
---
I was working on a small VSO gantt chart tile project with D3 and we needed to add tooltips for each bar. The problem came up when we need to display the tooltips in different directions depending on the position of the hovering bar as well as the container (VSO tiles in my example). Here is the solution for it:
First of all, there was a little bug (IMO) of d3tip itself that the classname of the tooltip didn't get reset. For example, once it has "n" class, it always is "n". If you change it to "w", it will append "w" to current classname. The fix is simple, open the d3 tip source code and look at tip.show function. 

{% highlight Javascript %}
if(className) nodel.node().className = className.apply(this, args);
{% endhighlight %}

OK, let's get to the real bussiness here. How do we calculate the direction for tooltips? Here is how the documentation suggests( we use callback function when initializing tip object):

{% highlight Javascript %}
tip.direction(function(d) {
  if (d === 'california') return 'w'
  if (d === 'new york') return 'e'
})
{% endhighlight %}

First of all, we need to know the width of the tooltip element. Why? Because if the tooltip is too big, and there is not enough room for showing it then it might be cut off. By default, d3tip will not allow you to access tooltip element inside callback function. That's why you need this piece of code:

{% highlight Javascript %}
args.push(node); //add this line right before d3tip calls our callback function
var dir = direction.apply(this, args)	
{% endhighlight %}

Our callback function will look like this:

{% highlight Javascript %}
tip.direction(function (data, index, y, nodel) {
	return tooltipUtils.calculateDirection(d, container, nodel); //here, I need container infomation about width, height ... nodel is the actual tooltip element
})	
{% endhighlight %}

We're almost there. Here is my calculateDirection function:

{% highlight Javascript %}
that.calculateDirection = function (d, chartContext, tooltipEl) {
	let coords = [];
	let bbox = d3.event.target.getBBox(); //get width, height of current svg element
	let tagName = d3.event.target.tagName;
	if (tagName === "rect" && bbox.x !== 0 && bbox.y !== 0) {
		coords.push(bbox.x);
		coords.push(bbox.y);
	}
	else if (tagName === "tspan") {
		coords = d3.transform(d3.select(d3.event.target.parentElement).attr("transform")).translate;
	}
	else {
		coords = d3.transform(d3.select(d3.event.target).attr("transform")).translate; //get transformed x,y
	}
	let centerX = coords[0] + bbox.width / 2;

	let centerY = coords[1] + bbox.height / 2;

	//calculate direction
	let direction = '';
	let halfWidthOfTooltip = tooltipEl.offsetWidth / 2;

	//North: we enough vertical and left horizontal rooms for tooltip
	if ((centerX + halfWidthOfTooltip < chartContext.width) && (centerY > tooltipEl.offsetHeight)) {
		if (centerX - halfWidthOfTooltip < 0 && (centerX + halfWidthOfTooltip < chartContext.width)) {
			//we don't have enough right horizontal room
			return 'e';
		}
		return 'n';
	}
	//East: we don't have enough vertical and horizontal rooms for tooltip
	else if ((centerX < halfWidthOfTooltip) && (centerY < tooltipEl.offsetHeight)) {
		return 'e';
	}
	//South: we don;t have enough vertical room for tooltip
	else if ((centerX + halfWidthOfTooltip < chartContext.width) && (centerY < tooltipEl.offsetHeight)) {
		return 's';
	}
	else if (coords[0] < tooltipEl.offsetWidth) {
		//this is when you hover to a long title, and there is no room in the left side, I don't know where to put it but rather put it in n than w
		return 'n';
	} else {
		return 'w';
	}
}
{% endhighlight %}

Happy coding!

Hung Cao