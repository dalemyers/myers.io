---
layout: single
title: Debugging Timer Triggers in Azure Functions
date: 2019-10-25 08:15:00
tags: [azure, functions]
---

It's relatively easy to figure out that VS Code is a great editor for developing
and debugging Azure Functions. You'll have also no doubt figured out that for
debugging HTTP trigger functions all you have to do is open a web browser or use
cURL or similar to trigger the function and be able to debug it. What about
timer triggers though?

It turns out that it's not that much harder. You just need to send a POST to
`http://127.0.0.1:7071/admin/functions/MyTimerTrigger` with a body of `{}` and
the content type set to `application/json`. 

That's all there is to it. If you miss out the body, or don't do a POST, you'll
get the function information back instead of triggering it.

