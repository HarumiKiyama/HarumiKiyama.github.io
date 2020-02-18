+++
title = "Waht is select"
date = 2020-02-19
publishDate = 2020-02-07T00:00:00+08:00
tags = ["UNIX", "CS"]
draft = false
creator = "Emacs 26.3 (Org mode N/A + ox-hugo)"
+++
什么是 select
<!--more-->

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">Table of Contents</div>

- [前言](#前言)
- [select 是什么](#select-是什么)
- [为什么需要 select](#为什么需要-select)

</div>
<!--endtoc-->


## 前言 {#前言}

select, poll 还有 epoll 是什么, 这个问题一直能够时不时的看到, 网上找了很多资料要么是语焉不详, 要么就是在解释这些api是如何处理IO多路复用的(`IO multiplexing`), 然后再对IO多路复用做一些以其昏昏使人昭昭的解释. 从某种角度来说这个问题也和某个函数式编程当中的经常有初学者问的问题, 即什么是 _monad_ 一样, 网上有很多种解释, 但是如果没有去使用过是完全不会明白 _monad_ 是什么的. 这也是工科和理科最大的一个区别吧, 站在工科的视角来看, 对于概念的定义并没有那么重要, 关键是如果讲这些概念应用到实际问题上去. 而理科的话则是需要死扣定义, 因为只有定义清晰了, 才可以创造出一个共同的语境(`context`), 这之后对于这个概念的讨论才有意义, 这方面最严谨的莫过于近现代数学公理化之后留下的遗产了.

select, poll 还有 epoll 是什么, 这个问题可以分成两部分, 第一部分, 这三个 api 在文档中是怎么定义的; 第二部分, 这三个 api 的实现是什么.
这里我只阐述这三个 api 是什么, 对于这三个 api 的实现是什么, 不做阐述, 特别的, 对于这三个 api 的文档特指, _POSIX Programmer's Manual_ 这可能和 _Linux Programmer's Manual_ 有所出入, 读者需要特别注意. 由于篇幅问题, 我这里将把对每个 api 的描述单独放到一篇文章中. 首先当然是排行第一的 _select_.


## select 是什么 {#select-是什么}

这个问题或许是最好回答的了, 如前所述, 代码的定义以及实现都是明明白白写在文档以及代码中的. 这里只做摘录以及必要的解释
select的规范是写在 _POSIX Programmer's Manual_ 中的, 如果读者使用的是内核为 _linux_ 的操作系统的话, 可以在 _shell_ 中输入 `man select` 或者 `man 3 select` 就会进入 select 的 _POSIX Programmer's Manual_ 页面 (**注意输入** `man 2 select` **会进入到** _Linux Programmer's Manual_ **中**). 我们注意到在该页面中, _DESCRIPTION_ 中写到 _refer to pselect()_, 于是我们再在 _shell_ 中运行 `man pselect` 转跳到 _pselect_ 的页面中.

在 _pselect_ 的页面中, 我们可以看到 _select_ 和 _pselect_ 的函数签名, _pselect_ 的签名如下

```c
int pselect(int nfds, fd_set *restrict readfds, fd_set *restrict writefds,
            fd_set *restrict errorfds, const struct timespec *restrict timeout,
            const sigset_t *restrict sigmask);
```

其中, `sigmask` 的作用为

> If sigmask is not a null pointer, then the pselect() function shall replace the signal mask of the caller by the set of signals pointed to by sigmask before examining the descriptors, and shall restore the signal mask of the calling thread before
> returning.

_select_ 的函数签名如下

```c
int select (int NFDS, fd_set *READ-FDS, fd_set *WRITE-FDS,
          fd_set *EXCEPT-FDS, struct timeval *TIMEOUT);
```

其中, `nfds` 参数为需要检验的文件描述符的范围, 在每一个 _xxFDS_ 中, 只校验 0 到 nfds - 1 的文件描述符. (**注意** `linux` **内核中并没有完全实现这个逻辑,详情可见** `man 2 select` **摘录如下**)

> According  to  POSIX, select() should check all specified file descriptors in the three file descriptor sets, up to the limit
> nfds-1.  However, the current implementation ignores any file descriptor in these sets that is greater than the maximum  file
> descriptor number that the process currently has open.  According to POSIX, any such file descriptor that is specified in one
> of the sets should result in the error EBADF.

fd\_set 为一个 _bit array_, 在 _POSIX Programmer's Manual_ 中没有明确定义, 但是在 _linux_ 的内核代码中有实现, 如下所示

```c
#define __FD_SETSIZE 1024
typedef struct {
  unsigned long fds_bits[__FD_SETSIZE / (8 * sizeof(long))];
} __kernel_fd_set;
```

然后在 _DESCRIPTION_ 中, 写明了 _pselect_ , _select_ 执行了什么功能, 以及 _pselect_, _select_ 分别有什么区别, 简单摘要如下

> The pselect() function shall examine the file descriptor sets whose addresses are passed in the readfds, writefds,
> and  errorfds  parameters  to see whether some of their descriptors are ready for reading, are ready for writing,
> or have an exceptional condition pending, respectively.
> The select() function shall be equivalent to the pselect() function, except as follows:
>
> -   For the select() function, the timeout period is given in seconds  and  microseconds  in  an  argument  of  type  struct
>     timeval,  whereas  for  the  pselect() function the timeout period is given in seconds and nanoseconds in an argument of
>     type struct timespec.
> -   The select() function has no sigmask argument; it shall behave as pselect() does when sigmask is a null pointer.
> -   Upon successful completion, the select() function may modify the object pointed to by the timeout argument.

简而言之, _select_ 和 _select_ 都是执行类似的功能的, 即对传入的参数 _readfds_, _writefds_, _errorfds_ 做校验, 判断是否处于 _ready_ 状态.
他们的差别在与

1.  timeout 的参数的类型不同
2.  _select_ 不需要传入 `sigmask` 作为参数
3.  在执行完之后, _select_ 会更新 `timeval` 的值, 新的值表示还有多少时间剩下来. 而 _pselect_ 则不会.

_select_ 和 _pselect_ 的差别就阐述到这里, 接下来就以 _select_ 为例, 来看看 _select_ 的行为是怎么定义的

> If the readfds, writefds, and errorfds arguments are all null pointers and the timeout argument is not a null pointer, the
> pselect() or select() function shall block for the time specified, or  until  interrupted  by  a  signal.  If  the  readfds,
> writefds, and errorfds arguments are all null pointers and the timeout argument is a null pointer, the pselect() or select()
> function shall block until interrupted by a signal.

当 _select_ 程序运行成功时

> Upon successful completion, the pselect() or select() function shall modify the objects pointed to by the readfds, writefds,
> and errorfds arguments to indicate which file descriptors are ready for reading, ready for writing, or have an error
> condition pending, respectively, and shall return the total number of ready descriptors in all the output sets. For each file
> descriptor less than nfds, the corresponding bit shall be set upon successful completion if it was set on input and the
> associated condition is true for that file descriptor.

当 _select_ 程序运行遇到错误时

> On failure, the objects pointed to by the readfds, writefds, and errorfds arguments shall not be modified. If the timeout
> interval expires without the specified condition being true for any of the specified file descriptors, the  objects  pointed
> to by the readfds, writefds, and errorfds arguments shall have all bits set to 0.
> −1 shall be returned, and errno shall be set to indicate the error.


## 为什么需要 select {#为什么需要-select}

select 在1983年被首次引入 BSD 中, 用于实现IO多路复用(`IO multiplexing`). 多路复用早期是一个通信行业的词汇, 指的是将多种信号合并通过同一媒介传播, 在计算机科学中多路复用指的是

> monitoring multiple file descriptors, waiting until one or more of the file descriptors
> become "ready" for some class of I/O operation (e.g., input possible)

在80年代服务器面对的客户端开始增多, 早期的一个线程对应一个输入信号的做法已经渐渐跟不上潮流, 于是一种更加高效的监听IO事件的 _system call_ 被开发了出来.
