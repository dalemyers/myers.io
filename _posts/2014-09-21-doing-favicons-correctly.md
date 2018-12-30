---
layout: single
title: Doing Favicons Correctly
date: 2014-09-21 12:00:00
tags: [favicon, http, internet, html]
---

Most web developers know of the existence of favicons. Once these were extremely
popular, but now with recent changes in browsers such as Chrome, which make them
less prominent, they are becoming less important. They are still useful however
when you have half a hundred tabs open and need a quick way of identifying which
tab is which. 

With such a wide spread "technology", one would assume that sites take full
advantage of it and do it the best way they possibly can. You would be wrong. 

The old style way of having a favicon was to have an ico file named
`favicon.ico` in the root directory of your website. Browsers would check for
the existence of this file and display it if it existed. This method is rather
inefficient as if there is not a favicon, then the browser still has to check.
The question is though, how many sites still use this outdated method with an
outdated image format? 

1. Google - Yes
2. Facebook - No
3. Youtube - No
4. Yahoo - No
5. Baidu - Yes
6. Wikipedia - No
7. Twitter - No
8. Amazon - Yes
9. LinkedIn - No
10. Qq - No

As we can see, 3 out of the top 10 sites world wide are still using this old and
outdated method. It is worth noting that all the sites which do not use the old
method, do provide it as a fallback. Each of these sites which still use the old
method are inefficient and promote bad practices. I'm especially surprised that
Google was a yes in this as I have come to think of Google following good
practices, especially since Youtube has done it correctly. 

So what is the new way? Basically you use a `link` tag in your HTML. The most
common one, which works in all browsers is as follows:

```html
<link rel="shortcut icon" href="http://example.com/myicon.ico" />
```

There are other variations which can be seen easily at
[this](http://en.wikipedia.org/wiki/Favicon#How_to_use) Wikipedia article. These variations let you do things like use PNG
images or GIFs. When combined with Javascript you can do all sorts of crazy
things like making [a game that is entirely played in a favicon](http://www.p01.org/releases/DEFENDER_of_the_favicon/).

The one thing that I've taken from this though, is that I really need to make a
favicon for this site... Any suggestions?
