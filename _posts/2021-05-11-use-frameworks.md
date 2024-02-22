---
layout: post
title: use_frameworks!
date: 2021-05-11 10:52:34
subtitle: use_frameworks!
categories: iOS
tags:
    - cocoapods
---

# use_framework!
默认使用启用

<!-- more -->

## OC 
- #use_frameworks!    
    - OC 库、swift 库都静态库 .a 的形式链接进主二进制
    - 使用 swift pod 库需要创建 bridge-header，没有 bridge-header 会报错 Undefined symbol

- use_frameworks!
    - OC 库、swift 库都动态库 .framework 的形式链拷贝到 Frameworks 文件夹，在运行时链接
    - 不需要创建 bridge-header
    - 添加了 `[CP] Embed Pods Frameworks` phase

## swift
- #use_frameworks!  
    - OC 库、swift 库都静态库 .a 的形式链接进主二进制
    - 需要创建 bridge-header，没有 bridge-header 会导致 OC 库无法使用

- use_frameworks!
    - OC 库、swift  .framework 的形式链拷贝到 Frameworks 文件夹，在运行时链接
    - 不需要创建 bridge-header
    - 添加了 `[CP] Embed Pods Frameworks` phase


use_frameworks! 决定了使用 framework 而不是静态库，use_frameworks! 的 :linkage 参数决定了 framework 是动态库还是静态库， 默认值是 :dynamic

use_frameworks! 会为 OC 库生成 modulemap 和 umbrella.h 文件 因此不用创建 bridge-header

# use_modular_headers!

在不使用 user_frameworks! 时，为 cocoapods 静态库创建 modulemap 和 umbrella.h。

一种使用场景是 swift 工程使用静态库的方式使用一个 OC 库。要使用这个 OC 库，一般是创建一个 bridge-header.h，在该文件中 #import, 然后在 swift 中使用。

还可以使用 use_modular_headers! 或 :modular_header => true 来为 OC 库创建 modulemap 和 umbrella.h。从而避免创建 bridge-header.h
```rb
    pod 'Alamofire'
    pod 'AFNetworking', :modular_headers => true
```
本文 [Demo](https://github.com/srv7/use_framework)