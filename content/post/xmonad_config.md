+++
title = "xmonad的配置"
date = 2019-03-31
publishDate = 2019-03-31T00:00:00+08:00
draft = true
creator = "Emacs 26.1 (Org mode 9.2.2 + ox-hugo)"
tags = ["CS"]
+++
个人的 `xmonad` 配置
<!--more-->

虽然从刚刚开始当程序员的那一年就开始用xmonad了，但是对于xmonad的配置却一直不怎么会，今天花了几个小时，总算是稍微学了一些皮毛。本来是有计划系统的看一下xmonad的源代码的，但是考虑到最近要看 `SICP` 还有努力转行 `java` 估计又要拖很久了。
把自己的配置附录一下吧。

```haskell
import XMonad
import XMonad.Util.EZConfig(additionalKeys)
import XMonad.Util.Run(spawnPipe)
import XMonad.Config.Desktop
import XMonad.Actions.SpawnOn

web = "web"
code = "code"
term = "terminal"
myWorkspaces = [web, code, term] ++ map show [4..9]

main = do
  xmproc <- spawnPipe "xmobar"
  xmonad $ desktopConfig {
    startupHook = do
      spawnOn web "firefox"
      spawnOn code "emacs"
      spawnOn term "termite"
    ,terminal = "termite"
    ,modMask = mod4Mask
    ,workspaces = myWorkspaces
    ,manageHook= manageSpawn
  } `additionalKeys` [
        ((mod4Mask, xK_Up), spawn "amixer set Master 2%+ unmute"),
        ((mod4Mask, xK_Down), spawn "amixer set Master 2%- unmute")
    ]

```
