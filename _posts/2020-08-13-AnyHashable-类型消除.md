---
layout: post
title: AnyHashable与类型消除
date: 2020-08-13 20:19:08
categories: swift
---


```swift
struct City: Hashable {
    let name: String
    let id: Int
}

var selected: [AnyHashable] = []

let city = City(name: "南京", id: 11)

selected.append(city)

let isCity = selected.first as? City
```

<!--more-->

## 问题

结构体 `City` 遵循 `Hashable` 协议。`selected` 是一个 `[AnyHashable]` 类型的数组。

从 Swift 标准库中可以知道 AnyHashable 是一个遵循 `Hashable` 协议的结构体。
虽然 `City` 和 `AnyHashable` 都遵循 `Hashable` 协议，但 `selected` 数组的类型是 `[AnyHashable]` 而不是 `[Hashable]`，那么问题就来了，`city` 是怎么能够添加到 `selected` 数组中的呢？

首先来看一下 `AnyHashable` 的定义

```swift
/// A type-erased hashable value.
///
/// The `AnyHashable` type forwards equality comparisons and hashing operations
/// to an underlying hashable value, hiding its specific underlying type.
@frozen public struct AnyHashable {
    public init<H>(_ base: H) where H : Hashable

    /// The value wrapped by this instance.
    ///
    /// The `base` property can be cast back to its original type using one of
    /// the casting operators (`as?`, `as!`, or `as`).
    ///
    ///     let anyMessage = AnyHashable("Hello world!")
    ///     if let unwrappedMessage = anyMessage.base as? String {
    ///         print(unwrappedMessage)
    ///     }
    ///     // Prints "Hello world!"
    public var base: Any { get }

    public static func != (lhs: AnyHashable, rhs: AnyHashable) -> Bool
}

extension AnyHashable : Equatable {

    /// Returns a Boolean value indicating whether two type-erased hashable
    /// instances wrap the same type and value.
    ///
    /// Two instances of `AnyHashable` compare as equal if and only if the
    /// underlying types have the same conformance to the `Equatable` protocol
    /// and the underlying values compare as equal.
    public static func == (lhs: AnyHashable, rhs: AnyHashable) -> Bool
}

extension AnyHashable : Hashable {
    public var hashValue: Int { get }
    public func hash(into hasher: inout Hasher)
}

extension AnyHashable : CustomStringConvertible {
    public var description: String { get }
}

extension AnyHashable : CustomDebugStringConvertible {
    public var debugDescription: String { get }
}

extension AnyHashable : CustomReflectable {
    public var customMirror: Mirror { get }
}
```
可以知道 `AnyHashable` 是用来做类型消除的，AnyHashable 可以表示任意遵循 `Hashable` 协议的类型，那么遵循 `Hashable` 协议的结构体 `City` 可以添加到 `[AnyHashable]` 类型的 `selected` 中也就说的通了。

话虽这么说，但除了都遵循 `Hashable` 协议之外， `AnyHashable` 与 `City` 是完全不同的类型，`selected.append(city)` 这行代码是如何通过编译的? `let isCity = selected.first as? City` 是如果转换成功的呢？

## 猜想

是不是编译器将 `append` 到 `selected` 之前将 city 封装成了 `AnyHashable` 呢？
是不是在 as? 的时候，使用 `AnyHashable` 的 `base` 来进行 `cast` 的呢？

## 验证

为验证上面的猜想，我们将上面的代码使用 `swiftc` 命令转为 `sil（Swift Intermediate Language）` 来一探究竟。

```shell
swiftc -emit-sil -Onone Foo.swift > Foo.swift.sil
```

得到的 `Foo.swift.sil` 文件内容如下

<script src="https://gist.github.com/srv7/0e5c88c115030c11fdd37b2fa3433323.js"></script>
 
从 66 ~ 74 行可以看出，在开始访问 `selected` 之前，执行了将 `city` 封装为 `AnyHashable` 的操作。

从 95 行、104~107 行、109~111 行，可以看出 `AnyHashabel` 向 `City` 的转换过程。

由此可以得出上面的猜想大致与实际情况是一致的。

## 关于 AnyHashable 的讨论

### AnyHashable 的实现
在 swift 的源码中可以到看 `AnyHashable` 的实现，是由 swift 和 cpp 共同实现的。这里有相关的讨论。

[Why isn’t AnyHashable implemented in pure Swift?](https://forums.swift.org/t/why-isn-t-anyhashable-implemented-in-pure-swift/33124)

### AnyHashable 的注释中存在的误导

```swift
/// The `AnyHashable` type forwards equality comparisons and hashing operations
/// to an underlying hashable value, hiding its specific underlying type.
```
```swift
let x = 1.0
let y = AnyHashable(x)
x.hashValue == y.hashValue  // false
```
注释中是 `AnyHashable` 将 comparisons 和  hashing 操作转发到 underlying hashable value，然而从上面的代码中可以看出并不是这样，如果是的话 x.hashValue == y.hashValue 将返回 true



```swift
/// Returns a Boolean value indicating whether two type-erased hashable
/// instances wrap the same type and value.
///
/// Two instances of `AnyHashable` compare as equal if and only if the
/// underlying types have the same conformance to the `Equatable` protocol
/// and the underlying values compare as equal.
```

```swift
let x: Float = 1.0
let y = AnyHashable(x)

let p: Double = 1.0
let q = AnyHashable(p)


// x == p // compile error
q.hashValue == y.hashValue // true
```
`AnyHashable` 类型的 y 和 q, 其 base 的 类型不同 分别为 `Float` 、`Double`，值为 1.0。但 y 和 q 的 `hashValue` 相同