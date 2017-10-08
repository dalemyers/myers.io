---
layout: single
title: Type validation in Python using decorators
date:   2013-10-25 13:42:12
tags: [python, types, decorators]
---

So in Python you generally do test driven development to make sure that you don't get weird type errors. Sometimes, however, it's good to be alerted immediately, so I've made a small decorator which can check for types of arguments as follows:

<script src="https://gist.github.com/dalemyers/7085125.js"></script>
