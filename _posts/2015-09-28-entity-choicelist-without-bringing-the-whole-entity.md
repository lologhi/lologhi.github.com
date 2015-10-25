---
layout:     post
title:      Entity choicelist without bringing the whole entity
date:       2015-09-28
summary:    Symfony forms have a nice entity type, but it will eat your memory if you have a too large entity. Here is the solution!
categories: symfony2
tags: symfony2 form choicelist entity
---

Symfony forms have a nice `entity` type, but it will eat your memory if you have a too large entity. Only answer I found while trying to find a fix for that was [this nice StackOverflow answer](http://stackoverflow.com/a/26234735/2989138), but nothing else.

The deal is to not bring the whole entity but only the needed information, like `id` and `name` to feed an HTML `select` tag.

{% gist bb900f061f8205841f6a %}
