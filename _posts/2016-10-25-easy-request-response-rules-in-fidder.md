---
layout: single
title: Easy Request Response Rules in Fiddler
date: 2016-10-25 11:52:00
tags: [fiddler, http, web]
---

Fiddler as a tool has gotten more and more advanced over time. With it's powerful scripting capabilities, there isn't much you can't do with it when dealing with HTTP requests. One of the problems with the scripting interface is the high barrier to entry. Fortunately, for a lot of things, there is an easier way. This post will focus on request response rules due to the fact that I'm often asked about them due my work on add-ins for Office. 

Sticking with the Office add-ins example, here is how you can rewrite a call to a script to one you have locally for easy debugging. (If you don't already have it, you can download Fiddler [here](http://www.telerik.com/fiddler))

### Setting up HTTPS Support

 Fiddler should do everything you need out of the box with one minor exception: SSL (or TLS hopefully) requests. In order to make it work with these requests, there is an extra step. Open `Fiddler Options` from the `Tools` menu, and select the `HTTPS` tab. Make sure that the `Decrypt HTTPS traffic` box is ticket and click `OK`. Fiddler can now intercept your HTTPS traffic and decrypt it. The problem is that you machine will, hopefully, reject these requests due to being [Man in the middled](https://en.wikipedia.org/wiki/Man-in-the-middle_attack). In order to ensure that your device will accept these certificates, the Fiddler certificate needs to be added to your store. This varies from device to device, but the gist of it is you will use the device which is making the connection you want to rewrite part of, browse to `X.X.X.X:8888` in your web browser, where `X.X.X.X` is the IP address of the machine running Fiddler, and chose to download the `FiddlerRoot certificate` towards the bottom of that page. More details about how to handle this for your device or browser can be found [here](https://www.google.co.uk/webhp?q=fiddler+decrypt+https+on+X#safe=off&q=fiddler+decrypt+https+on+X).

### Setting the AutoReponder

In the right hand pane, select the `AutoResponder` tab at the top. Make sure `Enable rules` and `Unmatched requests passthrough` are checked. Now, click `Add Rule`. At the bottom of the pane you will see a section named `Rule Editor`. For the first text field, you are going to enter `EXACT:https://the.url.of/my/script/here.js` , then for the bottom field, you have two choices. You can return a local file, or a response. 

For the local file, just select `Find a file...` in the dropdown, browse to the file you want to return, and select `Open`. Then click `Save` on the right hand side. 

If you want to return a different file hosted somewhere else (maybe a newer version of a script or similar), simple enter the URL of the file in the bottom field, and click `Save`

### Next Step

There isn't one. That's it. Make sure your device is set to use the machine running Fiddler as a proxy (remember it's on port 8888), and you are good to go. If it doesn't work quite as expected, make sure files aren't being returned from the cache.  


