---
layout: post
title: Lightweight Drag and Drop for iOS with CSS3 Translate
---

This post explores issues we experienced with a recent project, involving jQuery UI’s draggables and [how we solved it using CSS3 translate and javascript touch events](https://github.com/gbrassey/iDraggable "iDraggable").

In the midst of full production, we discovered an issue with iPad handling the combination of jQuery UI “draggables” and high quality images (for [using jQuery UI with iPad touch events](http://touchpunch.furf.com/ "Touch Punch")).

As production had already begun, we needed a shim to would work alongside what was already built, and replace the jQuery UI functionality on iOS devices.

We performed tests, sectioning off the drag and drops from the rest of the project and realized that, even one large background image severely affected the performance of the dragging animation on iOS devices. Scale that up to a production size eLearning platform and we suffered serious memory bleeds, causing Mobile Safari to crash instantly.

We googled far and wide but could find no solution. ([HTML5 drag and drop](https://developer.mozilla.org/en-US/docs/DragDrop/Drag_and_Drop "HTML5 Drag and Drop") would not fit the bill as it would require rebuilding everything we had done so far.)

And so we resolved to build a jQuery plugin and were pleasantly surprised to discover this undertaking was much simpler than first anticipated. Not only that, but our solution meant that, aside from changing the script which controlled these activities, we did not have to change any of the markup already written for dozens of pages.

## Development
This [blog post](http://popdevelop.com/2010/08/touching-the-web/ "Touching The Web") was a great jumping off point, it had done much of the hard work for us, showing us how to tie a touch event to a moving element. Despite being a great resource, the script animates with the “top” and “left” properties. While these work on all platforms, they use [a lot of CPU power](http://www.paulirish.com/2012/why-moving-elements-with-translate-is-better-than-posabs-topleft/ "Why Moving Elements With Translate() Is Better Than Pos:abs Top/left"), too much for the poor iPad. And so we updated the code to use CSS3 translate. This change was light and day. iOS webkit hardware accelerates CSS translates through the GPU and the performance improvement was significant.

Next we needed to add functionality to drop a “draggable” inside a “droppable”. This was done in two steps. First we added an initialization for “droppable” elements which would calculate the coordinates of the “droppable” and store these values in the data attribute of that element. Next we added an event handler for dropping an element, which finds if the last touch event occurred inside the bounds of a “droppable”. If this is the case, then we translate the “draggable” to sit on top of the “droppable”.

Along the way we added certain functionality specific to our project such as populating an object named “dragInput”, which contains the placement of any dragged items and can then be compared against another object which holds the correct matches for a quiz style drag and drop activity.

## Conclusion
Since integrating this into our project, I have tried to extend the plugin by adding mouse event listeners. There are limitations, such as dropping an element when the mouse escapes the bounds, despite the ‘mousedown’ event still being active. I have seen this behavior elsewhere. jQuery UI must use event listeners on the window to make up for this deficiency. Although I bemoan jQuery UI’s use of pre-CSS3 techniques, having tried to replicate the functionality with mouse events, I appreciate the depth of their project. The touch events were comparatively robust and behaved as expected.

CSS transforms are very powerful and although confusing at first glance, they give web developers exciting possibilities for creating native like experiences within browsers. By using transforms, our drag activities went from crashing the iPad to outperforming jQuery UI draggables on a desktop.

I hope this post proves helpful and I will continue to develop the plugin, as time and persistence permit.

Check out the github [here](http://github.com/gbrassey/iDraggable "iDraggable").

This post was originally appeared [here](http://www.themechanism.com/voice/2014/04/18/lightweight-drag-and-drop-for-ios-with-css3-translate/ "What The L!").