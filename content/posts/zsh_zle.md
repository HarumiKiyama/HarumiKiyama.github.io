+++
title = "zsh command line editting"
date = 2020-04-07
publishDate = 2020-04-04T00:00:00+08:00
tags = ["Workflow"]
draft = false
creator = "Emacs 26.3 (Org mode N/A + ox-hugo)"
+++
zshzle 介绍
<!--more-->
<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">Table of Contents</div>

- [前言](#前言)
- [zsh 的 keymap](#zsh-的-keymap)
- [zsh 自定义 keymap](#zsh-自定义-keymap)
- [后记](#后记)

</div>
<!--endtoc-->


## 前言 {#前言}

_zsh_ 在提供了很多的内置命令以及自己一套的配置系统之外还提供了非常方便的命令行编
辑功能. 所谓的命令行编辑, 就是可以在命令行中以 _Emacs_ 或者 _VIM_ 的 _keymap_
去进行编辑. 此外 _zsh_ 还提供了一些可以自定义 _keymap_ 的命令, 比如 _keybind_ 等


## zsh 的 keymap {#zsh-的-keymap}

在 _zsh_ 中所谓的 _keymap_ 是按键序列和 _zsh_ 命令行编辑命令的一个映射.

_zsh_ 一共提供了 8 个内置的 _keymap_, 分别是

-   emacs  EMACS emulation
-   viins  vi emulation - insert mode
-   vicmd  vi emulation - command mode
-   viopp  vi emulation - operator pending
-   visual vi emulation - selection active
-   isearch
    incremental search mode
-   command
    read a command name
-   .safe  fallback keymap

其中, _emacs keymap_ 为对 _Emacs_ 的命令的一个模拟, 比如 _C-f_ 会让光标向前移动
一格, _C-b_ 会让光标向后移动一格. _viins vincmd viopp visual_ 这些都是为了模拟
_VIM_ 的不同模式, 而不得不生成的 _keymap_. 个人建议纯用 _emacs_ 的 _keymap_ 就够
了. _zsh_ 默认状态下是启用 _Emacs_ 模式的, 只有在 _VISUAL_ 或者 _EDITOR_ 这两个
环境变量中包含有 _vi_ 才会启用 _VIM_ 模式.
_.safe keymap_ 为后备模式, 即只有在无法进入 _viins_ 等 _VIM keymap_ 或者 _emacs keymap_
才会进入 _.safe keymap_. 在这个 _keymap_ 中, 每个按键的意义 (除了 ^J 和 ^M 外)
都只是插入自身 (`self-insert`).
在 _emacs keymap_ 下按 _M-x_ 可以进入 _command keymap_, 按 _C-s_ 或者 _C-r_ 可以
进入 _isearch keymap_.


## zsh 自定义 keymap {#zsh-自定义-keymap}

_zsh_ 提供了工具 _bindkey_ 可以让用户对 _keymap_ 进行操作, 比如在特定的 _keymap_
中添加新的按键顺序与编辑命令的映射. 如果想要在 _emacs keymap_ 中加入 _C-g_ 对应
光标向前移动可以在命令行中或者 _zsh_ 的配置文件中加入 `bindkey -e ^G
forward-char`.
其中 _forward-char_ 是一个内置的 _widget_, 所谓的 _widget_ 是 _zsh_ 当中实现了一
些小功能的一个单位, 在 _kepmap_ 中和按键顺序对应的只能是 _widget_. 用 _emacs_ 中
的术语来解释的话就是类似 _interactive function_ 的东西.


## 后记 {#后记}

这里只是对于 _zsh command line editing_ 做了一个比较简略的介绍, 如果对这一部分比
较有兴趣可以通过 `man zshzle` 来阅读很多的资料.
