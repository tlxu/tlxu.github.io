---
layout: post
title:  "Zombie Process in Linux"
date:   2015-12-16
categories: Linux
---
" In UNIX System terminology, a process that has terminated, but whose parent has not yet waited for it, is called a zombie." -- From APUE

Example:

{% highlight c %}
#include <stdio.h>
#include <sys/types.h>

int main()
{
    printf("hello zombie demo.\n");

    pid_t pid;

    if ((pid = fork()) < 0) {
        printf("fork() error!!!\n");
    } else if (pid == 0) { /* child */
        printf("Enter child(%d)...\n", getpid()); 
        /* this is a child: dies immediately and becomes zombie */
        exit(0);
    } 

    /* parent */
    printf("Enter parent(%d)...\n", getpid());
    (void)getchar();

    return 0;
}
{% endhighlight %}

After compiling this program(gcc -o zomdemo) and running it(./zomdemo):

$ ./zomdemo
hello zombie demo.
Enter parent(8246)...
Enter child(8247)...

Don't hurry to press enter. Run the following fommands to check the zombie:

$ top |grep zombie
Tasks: 512 total,   1 running, 509 sleeping,   1 stopped,   1 zombie

$ ps aux |grep Z
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
fos        8247  0.0  0.0      0     0 pts/27   Z+   15:33   0:00 [zomdemo] <defunct>

$ pstree -ps 8247 |more
init(1)---lightdm(1533)---lightdm(1743)---init(2091)---screen(2828)---bash(2866)---zomdemo(8246)---zomdemo(8247)






