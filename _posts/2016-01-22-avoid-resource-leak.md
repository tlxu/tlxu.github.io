---
layout: post
title:  "How to avoid resource leak?"
date:   2016-01-22
categories: Linux
---
Just keep one single rule in mind: the one who creates a resource is responsible for destroying it.

A user of a resource should only use it and will NEVER destroy it.

In C/C++ programming, you can create a resource with new/malloc/open/fopen/dup...

The open/fopen/dup creates a new resource, but fdopen does NOT.

