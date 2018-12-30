---
layout: single
title: Websites Which Block Right-Click
date: 2014-10-13 12:00:00
tags: [we, web development, javascript, web]
---

We have all been there where we go to a website and try and right click on
something just to be greeted with an ugly popup stating that right click has
been disabled. Reasons for this seem to vary. Sometimes it is valid, such as
creating a game using a HTML canvas, but most of the time it is developers
believing that they can add "security" to the site by disabling right click.
After all, there is no way that  user can steal your images that way. 

Except that is completely wrong. Sometimes you just need to be able to right
click. There are multiple ways of disabling right click, but the most common way
is to use JavaScript and to reassign the `oncontextmenu` event. To get around
this simply open your browsers console and enter:

```javascript
document.oncontextmenu = new function(e){return true;}
```

Sometimes this isn't enough however and you may need to re-enable more
functions. Reason being that when you right click, the mouse down and up events
still fire and the click can be prevented there. Apply the following lines until
you find which works:

```javascript
document.onmousedown = new function(e){return true;}
document.onmouseup = new function(e){return true;}
document.body.oncontextmenu = new function(e){return true;}
document.onmousedown = new function(e){return true;}
document.onmouseup = new function(e){return true;}
```

Unfortunately, this list isn't fool proof. To do that would take far too much
time, if it is even possible. If these don't work, I suggest digging around in
the page source your self. 
