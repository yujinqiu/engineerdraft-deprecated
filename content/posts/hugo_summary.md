---
title: "Making Your Own Summary"
description: "Hugo allows you to use the top bit of your artical as a specific summary."
date: "2014-06-28"
categories: 
    - "template"
    - "hugo"
    - "tech"
---

Hugo allows you to specify where the summary stops - and allows the full markdown including shortcodes.

You just need to construct the first bit of your article so that it makes a nice summary and then end it with...

````
````
(of course dont include the space :) )

<!-- here is the real more... -->
<!--more-->

Everything after the more comment will not make it into the summary - sweet.
