---
layout: post
title: How to Build an Easy Embedabble Widget
---

Building with iframes is a fantastic way to create seamless, easy to implement, embeddable widgets. Once set up, creating multiple instances linking to your service is easy to do. Existing within a website, an *iframe* is like a window onto another website. Rather than forcing open another window, a user can interact with another service within the context of the website they have navigated to. The experience is smooth for the visitor, who will find the relevant service presented inline.

Over here at The Mechanism, we have been using iframes to integrate our custom bug tracking solution with client websites during user assisted testing. We needed an inconspicuous tool which would allow clients to seamlessly review work and submit bugs as they find them.

The requirements for this front end bug catcher:

1. Simple to embed and easy to implement across many projects
2. Sandboxed so that it doesn't cause any conflicts with the project DOM, JS or CSS
3. Simple architecture which works on all browsers
4. Responsive; it must work on all device sizes and form factors
5. Context Aware; diagnostic information will require knowledge of the parent document (the website under bug tracking )

Our first iteration of the bug tracking widget violated the second requirement. For our first prototype, we loaded a script and pulled in our view and styling files through JSONP (a method for circumventing Same-Origin Policy, you can read about it [here](http://www.themechanism.com/voice/2014/06/20/building-the-bugtrap-javascript-widget/ "Building The BugTrap JavaScript Widget")). This worked in our limited prototype but caused a few issues. First of all, we had to give our DOM elements verbose ID's and class names to ensure there would no conflict with the parent document. Secondly, we ran the risk of causing script conflicts with our dependencies. We used a script loader to minimize this risk, however we could never be sure. Finally, we were at the mercy of the stylesheets loaded by the parent which required us to write additional resets to ensure consistent appearance across projects. However, repairs like this are equivalent to bailing a sinking ship rather than repairing the leak.

So for our second iteration we converted the widget into an iframe. To do so we still had to find a way to get around the Same-Origin Policy. The [Same-Origin Policy](http://en.wikipedia.org/wiki/Same-origin_policy "Same-Origin Policy") restricts communication between two documents; the parent window and the iframe. Cross Document Messaging is a new addition to the HTML5 specification, which allows for simple string communication between documents. This is supported by most modern browsers, however, there are many hacks necessary for older browsers.

## easyXDM

[easyXDM](http://easyxdm.net "easyXDM") is a great library built to cross this great divide introduced by iframes. It uses the HTML5 `postMessage()` method when available and uses many fallbacks when necessary, ensuring the free flow of information between documents. For the developer, it exposes two protocols for data transfer. The first is a socket, which will send a string between the documents. This method requires us to parse and decipher the string before performing the relevant action on that information. The second option is an [RPC](http://en.wikipedia.org/wiki/Remote_procedure_call "Remote Procedure Call") (Remote Procedure Call), specifically JSON-RPC. JSON-RPC is a specification for calling functions from remote software, and for data to be returned. This allows for a much more dynamic interaction between our two documents where each process exists in their relative scope and can communicate as discreet functions would be expected. For our needs, the simpler option and the one we will be employing is the RPC protocol.

To implement easyXDM we must load our dependency and create an RPC instance, with the necessary proxy objects and method stubs. We will initiate this within a script loaded on the parent document. This script will embed our iframe and act as our gateway to the bug tracking widget.

On our parent document we will place an asynchronous script call to our remote script

	<script type="text/javascript">
		setTimeout(
			function(){
				var a = document.createElement("script");
				var b = document.getElementsByTagName("script")[0];
				a.src = "http://www.example.com/path/to/main.js";
				a.async = true;
				a.type = "text/javascript";
				b.parentNode.insertBefore(a,b)
		}, 1);
	</script>

In our script we will start by loading our easyXDM dependency

	// main.js
	var serverURL	= 'http://www.example.com/our-remote-service/',
		iframeFile 	= 'iframe.html',
		depends 	= {
			'easyXDM': serverURL + 'js/easyXDM.min.js'
		};

	Object.size = function(obj) {
		var size = 0, key;
		for (key in obj) {
			if (obj.hasOwnProperty(key)) size++;
		}
		return size;
	}

	var scriptCount = Object.size(depends);	// count of scripts required
	var scriptLoads = 0;	// count of script loaded

	for (var key in depends) {
		if (depends.hasOwnProperty(key)) {
			loadScript(key, depends[key], function() {
				scriptLoads++;
				if (scriptLoads === scriptCount) {
					main();
				}
			});
		}
	}

	function loadScript (dependency, src, callback) {
		// this function checks if the dependency is present.
		// it waits for load before executing the callback.
		if (window[dependency] === undefined) {	// if dependency is not present
			var scriptTag = document.createElement('script');
			scriptTag.setAttribute('type', 'text/javascript');
			scriptTag.setAttribute('src', src);
			if (scriptTag.readyState) {
				scriptTag.onreadystatechange = function () { // For old versions of IE
					if (this.readyState == 'complete' || this.readyState == 'loaded') {
						callback();
					}
				};
			} else { // Other browsers
				scriptTag.onload = callback;
			}
			(document.getElementsByTagName("head")[0] || document.documentElement).appendChild(scriptTag);
		} else {
			callback();
		}
	}

1. Lines 1-5 we are declaring a some variables that will be used later.
2. Lines 7-16 we extend the Object object with a method which will return the length of our depends array on line 15, along with a variable to hold an index value
3. Lines 18-27, we run a loop through the depends array and call a function `loadScript()`, which takes the name of our dependency, the url it can be found at and a callback which will be run once the dependency is loaded
4. Lines 29-49, our function which will test the presence of the dependency and load the script if it is not found. It uses various methods to ensure the script is loaded before running the callback function

Next we will create our RPC instance which will load the iframe

	// main.js
	.
	.
	.
	var iframeContainer = document.createElement('div');

	iframeContainer.style.position = 'fixed';
	iframeContainer.style.zIndex = 999;
	iframeContainer.style.bottom = 0;
	iframeContainer.style.left = 0;
	iframeContainer.style.top = "auto";
	iframeContainer.style.right = "auto";
	iframeContainer.style['max-height'] = '100%';
	iframeContainer.style['max-width'] = '100%';

	document.getElementsByTagName('body')[0].appendChild(iframeContainer);

	var rpc = new easyXDM.Rpc({
		remote: serverURL + iframeFile,
		container: iframeContainer,
		props: {
			id: 'bug-iframe',
			frameborder: '0',
			scrolling: 'no',
			marginwidth: '0',
			marginheight: '0',
			allowTransparency: 'true',
			style: {
				height: '100%',
				width: '100%',
				display: 'block'
			}
		}
	},
	{
		local: {
	        resizeiFrame: function (widthReq, heightReq, allowScroll) {
				var windowWidth = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth,
					windowHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;

				var width = (widthReq < windowWidth) ? widthReq : windowWidth;
				var height = (heightReq < windowHeight) ? heightReq : windowHeight;

				iframeContainer.style.width = width + 'px';
				iframeContainer.style.height = height + 'px';

				var sc = (allowScroll) ? 'yes' : 'no';
				document.getElementById('mech-bug-iframe').scrolling = sc;

	            return {
					x: width,
					y: height
	            };
	        },
	        parentInfo: function () {
	        	return {
	        		width: window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth,
	        		height: window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight,
	        		url: window.location.href
	        	};
	        }
	    }
	});

1. Lines 5-16; we are creating the iframes container with properties for it's layout within the parent documents DOM
2. Lines 18-34; our RPC instance with the address to find the iframe contents, a container to place the div in, and some properties to control it's appearance
3. Lines 34-64; is where we declare the methods we will be exposing to our iframe. `resizeiFrame()` and `parentInfo()` will allow us the adjust the size of the iframe and return diagnostic information respectively. They will be called from within our iframe


In our iframes markup we will load easyXDM and a shiv for older browsers without support for JSON, plus another .js file where we will instatiate our RPC connection.

	<!-- iframe.html -->
	<script src="/js/easyXDM.min.js" type="text/javascript"></script>
	<script type="text/javascript">
	    easyXDM.DomHelper.requiresJSON("/js/json2.js");
	</script>
	<script src="/iframe-main.js" type="text/javascript"></script>

In our iframe-main.js file, we will create another instance of easyXDM.Rpc and create stubs for our remote methods

	// iframe-main.js
	var rpc = new easyXDM.Rpc({},
	{
	    remote: {
	        resizeiFrame: {},
	        parentInfo: {}
	    }
	});

	.
	.
	.

	rpc.parentInfo(function(parentInfo) {
	    var diagObject = {
	        'width' = parentInfo.width,
	        'height' = parentInfo.height,
	        'url' = parentInfo.url
	    }
	});

1. Lines 2-8; we create our rpc object with the relevant stubs, referring to the remote methods
2. Lines 14-20; an example of how we call our remote function. Notice the anonymous function we pass to the remote function to return our requested data. This is an asynchronous function

Stay tuned for more on the Venus project to find out where it goes next. Dhruv Mehrotra will be back in a few weeks with a blog post going over some of the steps taken to set up the Ruby on Rails server behind Venus. And we will have meetup at our offices the second week of September. Hope to see you there!

This post originally appeared on [The Mechanism's Blog](http://www.themechanism.com/voice/2014/08/15/how-to-build-an-easy-embedabble-widget/ "How to Build an Easy Embedabble Widget").