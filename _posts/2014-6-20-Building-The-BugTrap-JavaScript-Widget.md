---
layout: post
title: Building The BugTrap JavaScript Widget
---

Over here at The Mechanism’s headquarters, Team Mechanism has been busy working on a better way to track bugs, code named: project:Venus. We want to make it easier for our clients to report bugs while reviewing projects and improve our workflow by allowing internal communication on a bug by bug basis.

The idea came to us as while working on another project. We realized we had the technology to build a swift prototype by leveraging tools that were already part of our arsenal. More on this in later posts. For now, we will focus on the front end javascript widget, “The Bug Trapper” if you will.

As an agency, we often have many projects in process so our bug tracker needs to be easy to implement across multiple projects and domains: we wanted to use a simple script tag which would be added to projects during test phases, with the project id included in the script ‘src’ GET parameters.

This posed a couple problems:

Scripts are not aware of the GET parameters from their requests
AJAX requests cannot load anything other than scripts from other domains

### 1. Javascript and Parameters Passed through GET Requests

We came across [this tip](http://loopj.com/2010/06/12/simple-way-to-extract-get-params-from-a-javascript-script-tag/ "Extract GET Params from a JavaScript Script Tag") for pulling the parameters from the script request. It actually has little to do with a GET request as the parameters are parsed by the script on the client side. By providing the name of the script, it searches the DOM for itself (javascript has no awareness of how it has been called or where it exists in the DOM). And then we do some regex magic to construct an object of key/value pairs. We have abstracted the code slightly from the original. Below is our script:

	// Extract "GET" parameters from a JS include querystring
	function getScriptTag(script_name) {
		// Find all script tags
    	var scripts = document.getElementsByTagName("script");
    	// Look through them trying to find ourselves
    	for(var i=0; i<scripts.length; i++) {
        	if(scripts[i].src.indexOf("/" + script_name) > -1) {
            	return scripts[i]
        	}
	    }
	    // No scripts match
	    return {};
	}
	function getParams(script_tag) {
	    // Get an array of key=value strings of params
	    var pa = script_tag.src.split("?").pop().split("&");

	    // Split each key=value into array, the construct js object
	    var p = {};
	    for(var j=0; j<pa.length; j++) {
	        var kv = pa[j].split("=");
	        p[kv[0]] = kv[1];
	    }
	    return p;
	}


### 2. Cross Domain AJAX Requests

For security purposes, AJAX requests only accept scripts from other domains. This is a problem for our widget which we’d like to build modularly, separating the script logic, markup and styling into separate files, while maintaing the simplicity of including a single script when creating new instances.

There is a work around and it involves turning the response from our server into JSONP. JSONP is JSON with Padding. Essentially our server response is turned into a JSON object and then gets wrapped in a function, which will get called on the client side and return an object containing our data.

Thank the heavens for jQuery. jQuery’s AJAX/getJSON method has baked in support for JSONP which will expect the callback function name to be a random string (an added layer of security), and will process the data, provided it gets the correct response from the server. On the client side, all we need to do is indicate we will be expecting the response to contain a callback function by adding “?callback=?” to our URL.


	var stylesheetURL = "http://example.com/getmystylehsheet?callback=?";
	$.getJSON(requestURL, function(data) {
	    $('head').append('<style>' + data + '</style>');
	}


On our rails server we route this request by wrapping the intended response in a function, the name of which is passed by jQuery as a parameter through the request. Below is the ruby on rails controller code to do this on our server:


	def getmystylesheet
	    css = File.read("path/to/stylesheet.css").to_s
	    json = {"css" => css}.to_json
	    callback = params[:callback]
	    jsonp = callback + "(" + json + ")"
	    render :text => jsonp,  :content_type => "text/javascript"
	end


We’re beginning a test cycle with the bug tracker on some internal projects and we will be rolling this out on upcoming client work. Hopefully, with feedback from our clients, we will continue this project with a view to scaling it. Stay tuned for future updates!

This post originally appeared on [The Mechanism's Blog](http://www.themechanism.com/voice/2014/06/20/building-the-bugtrap-javascript-widget/ "Building The BugTrap JavaScript Widget").