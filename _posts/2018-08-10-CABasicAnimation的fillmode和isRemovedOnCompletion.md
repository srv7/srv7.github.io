---
layout: post
title: CABasicAnimation的fillmode和isRemovedOnCompletion
date: 2018-08-10 11:15:46
categories: iOS
---

CABasicAnimation 的 fillMode 定义了在动画`active duration`结束后的行为状态，
> Determines if the receiver’s presentation is frozen or removed once its active duration has completed.

> Defines how the timed object behaves outside its active duration.Local time may be clamped to either end of the active duration, or the element may be removed from the presentation. The legal values are `backwards', `forwards`, `both' and `removed`. Defaults to `removed`.

<!-- more -->

在 Swift 4.2 以前 fillModel 的类型为 `String`


```
/* `fillMode' options. */

@available(iOS 2.0, *)
public let kCAFillModeForwards: String

@available(iOS 2.0, *)
public let kCAFillModeBackwards: String

@available(iOS 2.0, *)
public let kCAFillModeBoth: String

@available(iOS 2.0, *)
public let kCAFillModeRemoved: String

```


Swift 4.2 中 类型更改为 `CAMediaTimingFillMode`, 是一个 `struct`

```
public struct CAMediaTimingFillMode : Hashable, Equatable, RawRepresentable {

    public init(rawValue: String)
}

extension CAMediaTimingFillMode {

    
    /* `fillMode' options. */
    
    @available(iOS 2.0, *)
    public static let forwards: CAMediaTimingFillMode

    
    @available(iOS 2.0, *)
    public static let backwards: CAMediaTimingFillMode

    
    @available(iOS 2.0, *)
    public static let both: CAMediaTimingFillMode

    
    @available(iOS 2.0, *)
    public static let removed: CAMediaTimingFillMode
```

此属性常常配合另一个属性`isRemovedOnCompletion`配合使用,两者的组合效果如下图：
![](https://ws1.sinaimg.cn/large/b92f96b9gy1fu4fl6ufbpj20y2054abz.jpg)


