---
layout: post
title:  "Zombie Process in Linux"
date:   2015-12-16
categories: Linux
---
" In UNIX System terminology, a process that has terminated, but whose parent has not yet waited for it, is called a zombie." -- From APUE

{% highlight c %}
#include <stdio.h>
#include <sys/types.h>

int main()
{
    printf("hello fork() demo.\n");

    pid_t pid;

    if ((pid = fork()) < 0) {
            printf("fork() error!!!\n");
                } else if (pid == 0) { /* child */
                        printf("Enter child...\n");
                            } else { /* parent */
                                    printf("Enter parent...\n");
                                            (void)getchar();
                                                }

    printf("pid=%d\n", getpid());

    return 0;
}
{% endhighlight %}


$ top |grep zombie
Tasks: 512 total,   1 running, 509 sleeping,   1 stopped,   1 zombie

$ ps aux |grep Z
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
fos        8135  0.0  0.0      0     0 pts/27   Z+   15:08   0:00 [zomdemo] <defunct>

$ pstree -ps 8135 |more
init(1)---lightdm(1533)---lightdm(1743)---init(2091)---screen(2828)---bash(2866)---zomdemo(8134)---zomdemo(8135)



