---
layout: post
title:  "Get owner for a file"
date:   2016-03-18
categories: Linux, C
---

{% highlight C %}
#include <stdio.h>
#include <pwd.h>
#include <sys/stat.h>

int main(int argc, char *argv[])
{
    char *fname = argv[1];
    printf("file:%s\n", fname);
    
    struct stat info;
    stat(fname, &info);
    
    struct passwd *pw = getpwuid(info.st_uid);
    printf("user:%s\n", pw->pw_name);
    
    return 0;
}
{% endhighlight %}

        

