---
layout: post
title:  "Zombie Process in Linux"
date:   2015-12-16
categories: Linux
---
"In UNIX System terminology, a process that has terminated, but whose parent has not yet waited for it, is called a zombie." -- From APUE

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


A zombie is already dead, so you cannot kill it either by 'kill' or 'kill -9'. To cleanup a zombie, it must be waited on by its parent, so killing the parent should work to eliminate the zombie. After the parent dies, the zombie will be inherited by init, which will wait on it and clear its entry in the process table. If your program is spawning children that become zombies, you have a bug. Your program should notice when its children die and wait on them to determine their exit status.

$ kill -9 $(ps -A -ostat,ppid | awk '/[zZ]/{print $2}')


You can also get the ppid of a process with:

$ ps -xal |grep 8247 |awk '{print $4}' |head -1
8246

$ pstree -ps 8247 |more
init(1)---lightdm(1533)---lightdm(1743)---init(2091)---screen(2828)---bash(2866)---zomdemo(8246)---zomdemo(8247)


*Avoid zombie processes by calling fork twice*
A process whose parent has terminated will be inherited by init. The /init/ calls one of the/ wait/ functions to fetch the termination status. By doing this, init prevents the system form being clogged by zombies.

Example:
{% highlight c %}
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>

void do_something()
{
    int timeout = 10;
    while (timeout >0) {
        printf("pid=%d, ppid=%d\n", getpid(), getppid());

        sleep(1);
        timeout --;
    }
}


int safe_fork()
{
    pid_t pid;
    if ((pid = fork()) < 0) {
        printf("1st fork error\n");
        return -1;
    } else if (pid == 0) { /* first child */
        printf("Enter 1st child: pid=%d, ppid=%d\n", getpid(), getppid());

        if ((pid = fork()) < 0) {
            printf("2nd fork error\n");
            return -1;
        } else if (pid > 0) {
            /*sleep(5);*/
            printf("1st child exits: pid=%d, ppid=%d\n", getpid(), getppid());
            exit(0); /* parent from second fork == first child */
        }

        /*
         * We're the second child; our parent becomes init as soon
         * as our real parent calls exit() in the statement above.
         * Here's where we'd continue executing, knowing that when
         * we're done, init will reap our status.
         */
        printf("Enter 2nd child: pid=%d, ppid=%d\n", getpid(), getppid());

        do_something();

        printf("2nd child exits: pid=%d, ppid=%d\n", getppid(), getppid());        
        exit(0);
    }
    if (waitpid(pid, NULL, 0) != pid) { /* wait for first child */
        printf("waitpid for the 1st child error\n");
        return -1;
    }

    return 0;
}

int main()
{
    printf("hello double fork.\n");

    printf("Enter parent:pid=%d, ppid=%d\n", getpid(), getppid());

    int ret = safe_fork();

    /*
     * We're the parent (the original process); we continue executing,
     * knowing that we're not the parent of the second child.
     */
    printf("parent exits: ret=%d, pid=%d, ppid=%d\n", ret, getpid(), getppid());
    
    return 0;
}

{% endhighlight %}


fos@ubuntu:~/dev$ ./doublefork
hello double fork.
Enter parent:pid=11940, ppid=2866
Enter 1st child: pid=11941, ppid=11940
1st child exits: pid=11941, ppid=11940
Enter 2nd child: pid=11942, ppid=2091
pid=11942, ppid=2091
parent exits: ret=0, pid=11940, ppid=2866
fos@ubuntu:~/dev$ pid=11942, ppid=2091
pid=11942, ppid=2091
pid=11942, ppid=2091
pid=11942, ppid=2091
pid=11942, ppid=2091
pid=11942, ppid=2091
pid=11942, ppid=2091
pid=11942, ppid=2091
pid=11942, ppid=2091
2nd child exits: pid=2091, ppid=2091

fos@ubuntu:~/dev$ pstree -ps 2866 |more
init(1)---lightdm(1533)---lightdm(1743)---init(2091)---screen(2828)---bash(2866)

Note: you can uncomment sleep(5) in the above example and rerun this program to see the difference of the ppid of the 2nd child.
