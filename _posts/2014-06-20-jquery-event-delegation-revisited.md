---
layout: post
title: jQuery event delegation revisited
---

In my work, most of our JavaScript code handle user actions on differend kinds of lists. It is natural to use event delegation in this case to handle all actions using only one listener on list container. JQuery gives nice, cross-browser event delegation support using *$.delegate* or tripple parameter version of *$.on*. Unfortunately building something larger than a few lines of code using this style of event delegation can lead to unmaintainable spaghetti code.

Event delegation is only a kind of optimization providing easier and sometimes more performant way to handle events. Let's consider what do we expect from event handling.

##Following user actions

In most of cases when we listen DOM nodes we are not interested in doing listening itself but what we exacly want to do is **following user actions** to respond them in proper way. Below we have example of handling events on hypotetic product offers page.

```javascript

$('.offers-list')
  .on('click', '.availability', function(e){
    var $this = $(this)
        offerId = $this.data('id');
        
    e.preventDefault();
    
    //Do some stuff
  })
  .on('click', '.delivery-cost', function(e){
    var $this = $(this)
        offerId = $this.data('id');
    
    e.preventDefault();
    
    //Do some stuff
  })
  .on('click', '.add-favorite', function(e) {
    var $this = $(this)
        offerId = $this.data('id');
    
    e.preventDefault();
    
    //Do some stuff
  })
  .on('click', '.add-to-basket', function(e) {
    var $this = $(this)
        offerId = $this.data('id');
    
    e.preventDefault();
    
    //Do some stuff
  });
```

As you see code which handle only a few actions starts looking like spaghetti. The reason is that jQuery provides **low level** api which gives us a lot of control but from the other hand leads to mixing application logic with implementation details.

Let's try to analyse our **real** needs. As we see there is a few things we need to follow user:

- Functionality container, here $('.offers-list')
- User action definition
  - Action type (click, change etc.)
  - Action identificator (here jQuery selector)
- Action handler
- Action data (here offerId)

Now let's see what we have to do but we don't want:

- Listening to DOM (We would like listen to user than DOM)
- Have knowledge about DOM selectors (We only want action identificator)
- Low level work like getting data, *e.preventDefault* etc. (We are interested in effects - to have data, to don't have problem with default browser actions)

It turns out we strongly need something which operates on level of communication with user.

##$.controller comes to play

As I mentioned above it is reasonable to work on higher level than is given by jQuery. I decided to create missing $.controller plugin for jQuery, based on several interesting assumptions.

1. Remove all low level and repetitive work.
2. Minimize DOM relations
3. Store action identificator as DOM node attribute
4. Url-like action identificator format
5. Action handler data injection

The most interesting is url-like format of identificator. It gives great possibilities for namespacing and action characteristic data storage in compact form (instead of many data-xxx, data-yyy attributes). Let's see how it works. We should have piece of html:

```html
<span class="availability" data-action="offer/availability/11/22">
  Check availability
</span>
```

As you see we have action namespace "offer", action name "availability" and data - category id "11" and offer id "22". We can handle this using $.controller in the following way.

```javascript
$.controller('offer', {
  'availabilityClick': function(categoryId, offerId) {
      ...
    }
});
```

Creating controller we have to define namespace in first argument and handlers object in second argument. Namespace is first element from data-action attribute. Each handler name is defined as second argument of data-action attribute concatenated with action type. Controller passes data from data-action url-like identificator as next arguments.

Rewriting out initial example:

```javascript
$('.offers-list').controller('offer', {
  'availabilityClick': function(categoryId, offerId) {
      ...
    },
    'deliverycostClick': function(categoryId, offerId) {
      ...
    },
    'addfavClick': function(categoryId, offerId) {
      ...
    },
    'addtobasketClick': function(categoryId, offerId) {
      ...
    }
});
```

Basic implementation: https://github.com/pdorofiejczyk/playjs/blob/master/lib/controller.js

Be prepared
[Playjs is comming...](https://github.com/pdorofiejczyk/playjs)