---
layout: post
title:  "Tree view dropdown control for Visual Studio Online (VSTS) dashboard"
date:   2017-04-18 23:03:26 -0700
categories: vso-dashboard
---
Recently, I was working on a small project to create a VSO dashboard widget (tile). One of my tasks was to create a tree view dropdown for area path, which pretty much similar to the Area Path dropdown control in VSO workitem. If you guys come here somehow then I am pretty sure you already went through these pages:
[Add dashboard tutorial][add-dashboard-url]
[Tree view documentation][use-tree-view-url]
Therefore, I won't go through content in those pages in detail. Instead of that, I will go directly to the part that I've done to make it works.
This is the [Analytics service] [analytics-url] from Visual Studio that I was using to retrieve the data of area paths. Here is how the data looks like

{% highlight Json %}
{
	"@odata.context" : "https://{host}_odata/$metadata#Areas",
	"value" : [{
			"ProjectSK" : "xxxx-xx-xxxxx",
			"AreaSK" : "xxxx-xx-xxxxx",
			"AreaId" : "xxxx-xx-xxxxx",
			"AreaName" : "ParentName",
			"Number" : 11111,
			"AreaPath" : "ParentName",
			"AreaLevel0" : "ParentName",
			"Level" : 0
		}, {
			"ProjectSK" : "xxxx-xx-xxxxx",
			"AreaSK" : "xxxx-xx-xxxxx",
			"AreaId" : "xxxx-xx-xxxxx",
			"AreaName" : "ChildName",
			"Number" : 1111,
			"AreaPath" : "ParentName\\ChildName",
			"AreaLevel0" : "ParentName",
			"AreaLevel1" : "ChildName",
			"Level" : 1
		}
	],
	"@odata.nextLink" : "https://{host}/_odata/Areas?$filter=contains(AreaPath%2C'xxxx')&$orderby=AreaPath&$skip=2000"
}
{% endhighlight %}

Based on the response, they are using odata so you might need something like this to grab all the data
{% highlight Javascript %}
that.getAllAreaPaths = function () {
	var deferred = $.Deferred();
	var result = [];
	var url = "https://{host}/_odata/Areas?$filter=contains(AreaPath,'xxxx')&$orderby=AreaPath";
	function getAreaPaths(){
		$.get(url).done(function (response) {
			result = result.concat(response.value);
			if(response['@odata.nextLink']){
				url = response['@odata.nextLink'];                    
				getAreaPaths();
			}
			else{
				deferred.resolve(result);
			}
		}).fail(deferred.fail);
	}
	getAreaPaths();
	return deferred.promise();
};
{% endhighlight %}
So we already have all the data we need. Let's build a tree using it.
{% highlight Javascript %}

var data = response;

//let's build a tree, create dictionary of nodes
var dictionaryNode = {};

_.forEach(data, (node) => {
	//Iterate throught each node property    
	if (dictionaryNode[node['AreaPath']]) {
		//it's already created then that's fine for now                            
	}
	else {
		//we need to create new node                            
		var newNode = new TreeView.TreeNode(node['AreaName']);
		newNode.id = node['AreaId'];
		newNode.value = node['AreaPath'];
		//add to dictionary
		dictionaryNode[node['AreaPath']] = newNode;
		//Wait! we need to link to its parent, let take a look at it's parent name
		var parentAreaPathKey = "";
		for (var j = 0; j < node['Level']; j++) {
			var parentLevelPropertyName = "AreaLevel" + (j);
			parentAreaPathKey += node[parentLevelPropertyName] + "\\";
		}
		parentAreaPathKey = parentAreaPathKey.substring(0, parentAreaPathKey.length - 1); //remove last '\'  
		if (parentAreaPathKey) {
			dictionaryNode[parentAreaPathKey].add(newNode);
		}
	}
});

// Configure a few tree options
dictionaryNode['OSGS'].expanded = true; //hard coded OSGS for now
var treeViewOptions = {
	width: "100%",
	height: "100%",
	nodes: [dictionaryNode['OSGS']]
};
treeview = Controls.create(TreeView.TreeView, $('.areapath-wrapper'), treeViewOptions);
};
{% endhighlight %}

And of course we need to have html template for our control:
{% highlight Html %}
<div class="dropdown">
	<label>Area path</label>
	<div class="areapath-watermark">
		<span class="areapath-dropdown-watermark">Select area path</span>
		<span class="drop-icon bowtie-icon bowtie-chevron-down-light"></span>
	</div>
	<div class="warpper">
		<div class="areapath-wrapper popup-content-control">
		</div>
	</div>
</div>
{% endhighlight %}

There are a few things left like CSS or some event bindings. I will let you guys figure it out yourself cause it depends on your scope.
Example result
![Screenshot]({{ site.url }}/assets/vso-dashboard-create-areapath-post-1.PNG)

Happy coding!

Hung Cao

[add-dashboard-url]: https://www.visualstudio.com/en-us/docs/integrate/extensions/develop/add-dashboard-widget
[use-tree-view-url]:   https://www.visualstudio.com/en-us/docs/integrate/extensions/develop/ui-controls/treeviewo
[analytics-url]: https://www.visualstudio.com/en-us/docs/report/analytics/analytics-recipes
