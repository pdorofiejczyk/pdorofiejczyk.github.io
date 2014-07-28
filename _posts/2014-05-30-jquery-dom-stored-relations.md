---
layout: post
title: jQuery DOM stored relations
---

Creating jQuery reusable components, plugins or just writing piece of script code for website which contains many elements, we often face to problem with code heavily related to DOM. There are two common patterns of making such pieces of code:

* Structure-bound - End user of plugin have to recreate DOM structure
* Heavily configurable - End user have to write many lines of code during initialization of plugin

First of them is the worst because our plugin don't have any flexibility. The second is better but this is only way to move problem to end user.

Below we have 2 simple examples of expand button jQuery plugin.

##Structure-bound
```javascript
$.fn.expandButton = function() {
	$(this).on('click', function(e) {
  		var $this = $(this);
       	$this.prev('.expandable')
        		.toggleClass('expanded');
	});
}

//Usage
$('.expand').expandButton();
```
As You see we have strong structure binding defined by .prev() function. So to make it work You need piece of HTML where element with class .expandable is before click target. We can't have our expandable element anywhere else.

##Heavily configurable
```
$.fn.expandButton = function(config) {
	$(this).on('click', function(e) {
		var $this = $(this);
       	config.getExpandTarget($this)
        		.toggleClass(config.expandClass);
	});
};

//Usage
var $expand = $('.expand');
$expand.expandButton({
    'getExpandTarget': function($target) { 				$target.prev('.expandable');
    },
    'expandClass': 'expanded'
});
```
In this case we can configure our plugin during initialization. We decide about relation by passing getExpandableTarget function. Better, but still we don't have full flexibility. One plugin initialization is no enought to handle multiple relation types. What's more we mix informations about html markup with business logic.

##Relations

Both examples shows the problem with defining relations between DOM nodes. The question is where and when we should define that relations. As relations definitions refers directly to DOM nodes identificators like ids and classes, informations about relations should be stored in the same place.

The idea is to add relations definitions to DOM nodes inside custom attributes. jQuery gives special function data() which gives possibility for easy extraction of data-* attributes so we can use this kind of attributes.

Let's consider again our expand button example.

```html
<p class="expandable e1" data-expanded-class="expanded">
  Lorem ipsum [...]
</p>
<div class="expand" data-relation=".expandable.e1">Expand</div>
<p class="expandable e2" data-expanded-class="expanded">
  Lorem ipsum [...]
</p>
<div class="expand" data-relation=".expandable.e2">Expand</div>
```

In above code we have relations defined in data-relation attribute. Each expand button have related expandable box. We have created relation tree, independent of DOM tree. We can use this relations to create more flexible plugins.

```javascript
$.fn.expandButton = function() {
	$(this).on('click', function(e) {
		var $this = $(this),
        	$related = $($this.data('relation')),
            expandClass = $this.data('expandClass');
            
            $related
                .toggleClass(expandClass);
	});
};

//Usage
$('.expand').expandButton();
```

We found simple and really powerfull pattern for building reusable, flexible components. It gives huge possibilities for building clean and not related to DOM Javascript code.

Original jsfiddle example [http://jsfiddle.net/MgUnS/1/](http://jsfiddle.net/MgUnS/1/)