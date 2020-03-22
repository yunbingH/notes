    fork大家可能都比较熟悉，调用一次返回2次，返回pid>0为父进程，pid=0为子进程。一直对fork方法如何返回两次有些疑惑，函数调用一次通常只返回一个结果，fork调用怎么会返回2次？之前查过原因，时间久了有些忘了，今天上网又查了下，其实大家说的“返回两次”在表达上时不清楚的。fork实际上并不是执行两次，它依然返回了一次，准确地说是一次多一点，只是OS对fork进行的操作使得我们看起来它返回了两次而已。
    
    要搞清楚fork的执行过程，就必须先讲清楚操作系统中的“进程(process)”概念。一个进程，主要包含三个元素： 
    1） 一个可以执行的程序； 
    2）和该进程相关联的全部数据（包括变量，内存空间，缓冲区等等）； 
    3）程序的执行上下文（execution context）。 
    
    不妨简单理解为，一个进程表示的，就是一个可执行程序的一次执行过程中的一个状态。操作系统对进程的管理，典型的情况，是通过进程表完成的。进程表中的每一个表项，记录的是当前操作系统中一个进程的情况。对于单 CPU的情况而言，每一特定时刻只有一个进程占用 CPU，但是系统中可能同时存在多个活动的（等待执行或继续执行的）进程。 
    
    一个称为“程序计数器（program counter, pc）”的寄存器，指出当前占用 CPU的进程要执行的下一条指令的位置。 
    
    当分给某个进程的 CPU时间已经用完，操作系统将该进程相关的寄存器的值，保存到该进程在进程表中对应的表项里面；把将要接替这个进程占用 CPU的那个进程的上下文，从进程表中读出，并更新相应的寄存器（这个过程称为“上下文交换(process context switch)”，实际的上下文交换需要涉及到更多的数据，那和fork无关，不再多说，主要要记住程序寄存器pc指出程序当前已经执行到哪里，是进程上下文的重要内容，换出 CPU的进程要保存这个寄存器的值，换入CPU的进程，也要根据进程表中保存的本进程执行上下文信息，更新这个寄存器）。 

再来说说fork，为什么会返回两次？

    当程序执行到下面的语句： pid=fork();  
    由于在复制时复制了父进程的堆栈段，所以两个进程都停留在fork函数中，等待返回。因此fork函数会返回两次，一次是在父进程中返回，另一次是在子进程中返回，这两次的返回值是不一样的。
    fork调用的一个奇妙之处就是它仅仅被调用一次，却能够返回两次，它可能有三种不同的返回值：
　   1）在父进程中，fork返回新创建子进程的进程ID；
       2）在子进程中，fork返回0；
       3）如果出现错误，fork返回一个负值。

    我们可以通过fork返回的值来判断当前进程是子进程还是父进程。引用一位网友的话来解释fork函数返回的值为什么在父子进程中不同。“其实就相当于链表，进程形成了链表，父进程的fork函数返回的值指向子进程的进程id, 因为子进程没有子进程，所以其fork函数返回的值为0.
    
    调用fork之后，数据、堆、栈有两份，代码仍然为一份但是这个代码段成为两个进程的共享代码段都从fork函数中返回。当父子进程有一个想要修改数据或者堆栈时，两个进程真正分裂。

   子进程代码是从fork处开始执行的，为什么不是从#include处开始复制代码的？这是因为fork是把进程当前的情况拷贝一份，执行fork时，进程已经执行完了int count=0; fork只拷贝下一个要执行的代码到新的进程。

看一个例子：

#include <unistd.h>
#include <sys/types.h>

main ()  
{  
        pid_t pid;  
        printf("fork!");    // printf("fork!\n"); 
        pid=fork();  

        if (pid < 0)  
                printf("error in fork!");  
        else if (pid == 0)  
                printf("i am the child process, my process id is %d\n",getpid());  
        else  
                printf("i am the parent process, my process id is %\/n",getpid());  
}
结果是  
[root@localhost c]# ./a.out  
fork!i am the child process, my process id is 4286  
fork!i am the parent process, my process id is 4285 

但改成printf("fork!\n");后，结果是 
[root@localhost c]# ./a.out 
fork!  
i am the child process, my process id is 4286  
i am the parent process, my process id is 4285 

为什么只有一个fork!打印出来了？上一个为什么有2个？

APUE的P143说：
fork之前调用了printf一次，当fork以后，该行数据仍在缓存中，然后父进程数据空间复制到子进程中，该缓存的数据也被复制到子进程中。

在printf()与fork()之间再加一个函数fflush(0)，就可以看出是怎么加事了。

```
#include <stdio.h>
int main(void)
{
printf("hello"）；
fflush(stdout);
fork();
exit(0);
}
```


这样只输出hello 
原因是：stdin,stdout,stderr都是行缓冲 

清空缓冲就不会有两个输出了：setbuf(stdout, NULL)或setvbuf(stdout, NULL, _IONBF, 0)
------------------------------------------------
