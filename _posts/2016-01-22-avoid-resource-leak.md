---
layout: post
title:  "How to avoid resource leak?"
date:   2016-01-22
categories: Linux
---
Just keep one single rule in mind: the one who creates a resource is responsible to destroy the resource.

In C/C++ programming, you can create a resource with new/malloc/open/fopen/dup...

With open/fopen/dup you create a new resource, but fdopen does NOT.

