+++
title = "zsh overview"
date = 2020-03-13
publishDate = 2020-03-09T00:00:00+08:00
tags = ["Workflow"]
draft = false
creator = "Emacs 26.3 (Org mode N/A + ox-hugo)"
+++
zsh 概览
<!--more-->

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">Table of Contents</div>

- [前言](#前言)
- [zsh 的历史](#zsh-的历史)
- [zsh 的配置文件](#zsh-的配置文件)
- [Options of zsh](#options-of-zsh)
- [zsh 的内置命令](#zsh-的内置命令)

</div>
<!--endtoc-->


## 前言 {#前言}

从几年前开始用 _archlinux_ 时就在用 _zsh_ 作为默认的 _shell_ (因为 _zsh_ 是
_archlinux_ 的安装镜像中默认的 _shell_), 但是因为一直都是用的 _oh-my-zsh_, 这就
类似于 _Ubuntu_ 把 _linux/的很多细节隐藏了一样, /oh-my-zsh_ 也把很多 _zsh_ 的细
节隐藏了. 这几天刚好有点时间, 就把 _zsh_ 的文档
[^fn:1]看了一下, 这里也算是
做个记录吧.


## zsh 的历史 {#zsh-的历史}

保罗·弗斯塔德 (`Paul Falstad`) 在 1990 于普林斯顿大学求学期间编写了 _zsh_ 的初版,
当时只是为了增强 _Bourne Shell_ 的性能写的一个项目, 这两者之间的关系比较类似与
_VIM_ 和 _VI_ 的关系. 在看到了普林斯顿教授邵中的登录名 _zsh_ 之后, 保罗就自作主
张的把自己写的 _shell_ 的命名为 _zsh_.


## zsh 的配置文件 {#zsh-的配置文件}

如果把 _zsh_ 设为默认的 _shell_, 那么在开机到登陆完成为止, 配置文件的加载顺序为
`/etc/zshenv` (注之后的行为会根据 _RCS_ 和 _GLOBAL\_RCS_ 的不 同而不同, 具体会在
下一小节讲解), `$ZDOTDIR/.zshenv`, `/etc/zprofile`, `$ZDOTDIR/.zprofile`,
`/etc/zlogin` (这里以及之后的文件都是在登陆密码用户名校验完成之后才记载的),
`$ZDOTDIR/.zlogin`. 其中, 如果 `$ZDOTDIR` 没有设置的话, 默认使用 `$HOME` 的值.

在登陆之后, 如果打开一个 _terminal_ 的话, 那么配置文件的加载顺序为,
`/etc/zshenv`, `$ZDOTDIR/.zshenv`, `/etc/zshrc`, `$ZDOTDIR/.zshrc`. 这里我们可以
看到, _zshenv_ 这是一个非常的重要的文件, 应该尽可能的保持精简, 并且如果是需要全
局配置的一些环境变量的话, 最好写在其中. 此外值得一提的是, 在推出登陆的时候, 会加
载 `/etc/zlogout`, `$ZDOTDIR/.zlogout`.


## Options of zsh {#options-of-zsh}

这里插一句题外话, 因为实在不知道 _options_ 应该怎么翻译, 这里先暂且统一称为配置吧.
存在一些配置可以让用户对 _zsh_ 定制化(`customization`), _zsh_ 对于这些配置的大小
写不敏感, 并且会无视变量中的下划线, 比如 `allexport` 和 `A__lleXP_ort` 对于
_zsh_ 来说是一样的.
配置可以写在上一节所提到的配置文件的任意位置, 举个例子, 在 _zsh_ 中, 默认是开启
_auto\_cd_ 的也就是说, 在 _zsh_ 的 _interactive shell_ 里面, 直接输入文件夹的路径
就可以进入该文件夹, 那么该如何关闭这一行为呢, 很简单, 在 `.zshrc` 文件的末尾加上
`unsetopt autocd` (这里可以在其他任何合法的配置文件中输入, 之所以是最后一行, 是
因为如果使用了 _oh-my-zsh_ 的话, 在 `source $ZSH/oh-my-zsh.sh` 这一行之前添加的
话有可能会导致配置失效.) 另, 具体的配置条目可以看官方的文档.


## zsh 的内置命令 {#zsh-的内置命令}

_zsh_ 提供了很多有用的内置命令
[^fn:2](`build-in
commands`), 很多内置命令都实现了 _POSIX Programmer' Manual_ 里面的一些功能, 但是
又多多少少有点出入. 举个例子, 在 _POSIX Programmer's Manual_ 里面, 对于 _cd_ 的
使用方法是这样描述的.

> cd [−L|−P] [directory]
> cd −

但是在 _zsh_ 当中却可以这样调用 `cd -2` 甚至这样调用 `cd 2`, 这个原因在于 _zsh_
内置的 _cd_ 命令可以对目录栈做更多的操作. `cd -2` 和 `cd 2` 的行为是一致的就是进
入在目录栈上的第三个元素所对应的目录.

[^fn:1]: <http://zsh.sourceforge.net/Doc/Release/index.html#Top>
[^fn:2]: <http://zsh.sourceforge.net/Doc/Release/Shell-Builtin-Commands.html#Shell-Builtin-Commands>
