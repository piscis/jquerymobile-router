jQueryMobile-Router
================================================================================

jQuery Mobile router is a plugin for jQuery Mobile to enhance the framework
with a client side router/controller that works with jQuery Mobile events
(pagebeforecreate, pagecreate, pagebeforeshow, pageshow, pagebeforehide, pagehide,
pageremove).

In addition, it extends jQM with a client-side parameters in the hash part of the url
(yay!!!)

The jQuery Mobile router javascript file must be loaded before jQuery Mobile.

This plugin can be used alone or (better) with Backbone.js or Spine.js, because it's
originally meant to replace their router with something integrated with jQM.

What's new in the latest versions
=====================
* Support for a different syntax defining your routes
* Added a nice getParams() function to actually 'parse' parameters in the hash and get
a simple object to play with them.
* Support for the pageinit event.
* Bugfixes to support events for the first displayed page.
* Default handler support
* The original jquery mobile event is passed down to route handlers

Upgrade notes
=====================
0.5 to 0.6
	You can define a default handler in the "conf" object. This will be called when
	no matching routes are found in the set you've provided

0.3 to 0.4
	The main javascript file has been renamed to jquery.mobile.router.js

0.2 to 0.3
	There's no need to use the data-params="true" anymore in your anchors since hash params
	are enabled by default.

**The reuseQueriedAjaxPages extension was removed since it wasn't so useful and
a similar behaviour can be achieved with the new jquery mobile caching mechanism
(but if you need it please mail me).**


The router/controller
=====================

Whenever jQuery Mobile changes the url (usually the fragment part) the router checks
if that particular url matches one of your routes and calls the handler you've
provided with a bunch of useful arguments.

When you define a route, you'll provide:
* a regular expression to test the url/hash against
* an handler (a function)
* when your handler must be called (for example, you may decide to setup a route only when the pagecreate and pagebeforeshow jQM events are dispatched)

The plugin exports a class in $.mobile.Router and you can instantiate your routers
with the following arguments:

`var approuter=new $.mobile.Router(myRoutes, myHandlers, options);`

* myRoutes is an object or an array defining your routes
* myHandlers is an object with your function handlers
* options is an object with a certain configuration (see below)


Here are a few examples:

```javascript
var router=new $.mobile.Router([
        { "/index.html": { events: "i", handler: "index" } },
        { "/events.html(?:[?](.*))?": { events: "i", handler: "events" } },
        { "/eventDetail.html(?:[?](.*))?": { events: "i", handler: "eventDetail" } },
        { "/accomodations.html(?:[?](.*))?": { events: "i", handler: "accomodationsTaxonomy" } },
        { "/accomodationList.html(?:[?](.*))?": { events: "i", handler: "accomodations" } }
], ControllerObject, { ajaxApp: true} );

var router=new $.mobile.Router([
        { "#ticketPlanner([?].*)?": "ticketPlannerShown" },
        { "#ticketConfirm": "ticketConfirmShown" },
        { "#ticketDetail([?].*)?": { handler: "ticketDetailPage", events: "bs,h" } },
        { "#ticketHistory": "ticketHistoryShown" },
        { "#map": { handler: "mapHidden", events: "bh" } },
        { "#map": { handler: "mapShown", events: "s" } }
], ControllerObject);
```

**----- IMPORTANT (no kidding!) -----**

* By default, the router will match the routes against the hash part of the url.
	  If you need the FULL PATH (pathname+search+hash), please set the "ajaxApp" configuration
	  parameter to TRUE (see below)

* If you're using a multipage template in your jquerymobile application, the first displayed page is quite an "anomaly" in the navigation model, because, even if it has its own id, this
	  is not reflected in the hash. This doesn't happen for other pages. This "empty hash" problem
	  may come into play when the back button is used by the user.
	  Since writing an "empty" regular expression such as "^$" to match this page seems really
	  strange, the router will accept *only* a route with the page id, for example "#foobar"

myRoutes object
---------------
`myRoutes` supports the following formats:

1. This one binds a certain route to the pagebeforeshow event and calls the handler, which can be an inline function or a function name (a string) that must be defined
	  in the myHandlers object

```javascript		
		{
			"regularExpression": "handlerName",

			/* or */
			
			"regularExpression": function(){
				// your inline function here ...
			}

			/* or */
			
			"regularExpression": someobject.functionReference
		}
```

2. This is the full syntax to specify various jQM events you want your route to be
	  bound to.
	  The object defines an "handler" property with a string value: this is the name
	  of a function defined in the myHandlers object. Again, you may also put an inline
	  function in the "handler" property instead of a string.
	  The object also defines an "events" property with a string value: this is a list
	  of (shortened) jQM events, separated by a ",". Your route will be called only when
	  these events are fired. 

```javascript	  
		{
			"regularExpression": { 
				handler: "handlerName",
				events: "bc,c,bs,s,bh,h"
			},

			/* or */

			"regularExpression": { 
				handler: function(){ ... },
				events: "bc,c,bs,s,bh,h"
			},
		}		
```

Please refer to the following schema to understand event codes (it's really straightforward)

```javascript
		bc	=> pagebeforecreate
		c	=> pagecreate
		i	=> pageinit
		bs	=> pagebeforeshow
		s	=> pageshow
		bh	=> pagebeforehide
		h	=> pagehide
		rm	=> pageremove
```

* The above syntax, however, doesn't let one use the same regular expression to call different handlers (this is due to the fact that the regexp is a key into an hashmap,
	  so it must be unique). If you need, for instance, to call the function "foo" when a
	  certain page has been shown, and the function "bar" when the same page has been hidden,
	  you could use the following syntax:

```javascript
		var approuter=new $.mobile.Router([
			{ "#certainPage": { handler: "foo", events: "s" } },
			{ "#certainPage": { handler: "bar", events: "h" } }
		], {
			foo: function(...){...},
			bar: function(...){...}
		}, options);
```

By using an array, you can specify the **SAME REGULAR EXPRESSION** multiple times, but for **DIFFERENT EVENT TYPES**.


This is an example of a common `myRoutes` object:

Syntax 1:

```javascript
	{
		"#localpage(?:[?/](.*))?": {
			handler: "localpage", events: "bs,bh"
		},

		"ajaxPage.html(?:[?](.*))?": {
			handler: "ajaxPage", events: "c,bs"
		}
	}
```

Syntax 2:

```javascript
	[
		{ "#localpage(?:[?/](.*))?": { handler: "localpage", events: "bs,bh" } },

		{ "ajaxPage.html(?:[?](.*))?": { handler: "ajaxPage", events: "c,bs" } }
	]
```


myHandlers object
-----------------
There isn't much to say about this object. Simply provide the function handlers you've
specified in the myRoutes object.

For example:

```javascript
	{
		handlerName: function(eventType, matchObj, ui, page, evt){
			// your code here
		}
	}
```

Your handlers will be called with the following arguments:

* eventType: the name of the jQM event that's triggering the handler (pagebeforeshow,
	pagecreate, pagehide, etc)

* matchObj: the handler is called when your regular expression matches the current
	url or fragment. This is the match object of the regular expression.
	If the regular expression uses groups, they will be available in this object.
	Cool eh?

* ui: this is the second argument provided by the jQuery Mobile event. Usually holds
	the reference to either the next page (nextPage) or previous page (prevPage).
	More information here: (http://jquerymobile.com/demos/1.0/docs/api/events.html)[http://jquerymobile.com/demos/1.0/docs/api/events.html]

* page: the dom element that originated the jquery mobile page event

* evt: the original event that comes from jquery mobile. You can use this to
	prevent the default behaviour and, for instance, stop a certain page from being
	removed from the dom during the pageremove event.


Public methods
--------------

Router objects have the following public methods:

*`add(myRoutes,myHandlers)`:
	You can dynamically add routes on an already instantiated router.
	The myRoutes and myHandlers objects were already described above.

*`destroy()`:
	Unbind events and deactivate this router instance

*`getParams(hashPartOfTheUrl)`:
	Returns an object with the parameters encoded in the url or null
	if nothing's found.
	For instance, if you have something like:
				`#detail?id=3&foo=bar`
			and call `routerInstance.getParams("?id=3&foo=bar")`
			you'll get:
```javascript				
				{
					id: "3",
					foo: "bar"
				}
```

**There's an example under examples/backbone-example !**


jQM Configuration
==================

jQuery Mobile Router supports the following parameters:

*`ajaxApp`: tells the plugin to use the full page path for its matches instead of
		 the hash part of the url


*`defaultHandler`: a function reference or a function name to be called when no matchin
		route is found

*`defaultHandlerEvents`: the defaultHandler will be called for these events, if
		no routes are matched. Please note that THESE EVENTS MUST BE DEFINED 
		AT LEAST ONCE IN ANOTHER ROUTE, otherwise the defaultHandler won't be executed.
		That is to say, if your routes are defined for "pagebeforeshow" and "pageshow"
		events only, a defaultHandler for the "pagehide" event won't work!


You can pass an object with the above properties to the single router instance or set it
globally with this code (must be used BEFORE loading jquery.mobile.router.js):

```javascript
	$(document).bind("mobileinit",function(){
		$.mobile.jqmRouter={
			ajaxApp: true
		};
	});
```

Notes on jQM router
==============

Have you ever wanted client-side parameters in the hash part of the url in jQuery Mobile?

Well, jquery mobile router automatically enables this feature for you, using an official
*"hack"* provided in the jQM documentation.

* This hack isn't quite different from the one previously used by this plugin. But since
there's something 'official', we'll be guaranteed that our code will be supported in future
releases and I'm extremely happy about this.


For bugs, comments, patches and requests mail me!

License
==============

Copyright 2011 (c) Andrea Zicchetti

You may use jQueryMobile-Router under the terms of either the MIT License or the GNU General Public License (GPL) Version 2.

You don’t have to do anything special to choose one license or the other and you don’t have to notify anyone which license you are using. You are free to use a jQueryMobile-Router in commercial projects as long as the copyright header is left intact.

**For more information see:**

* [MIT License](http://github.com/azicchetti/jquerymobile-router/blob/master/MIT-LICENSE.txt) [(More Information)](http://en.wikipedia.org/wiki/MIT_License)
* [GPL](http://github.com/azicchetti/jquerymobile-router/blob/master/GPL-LICENSE.txt) [(More Information)](http://en.wikipedia.org/wiki/GNU_General_Public_License)