---
layout: post
published: false
title: Swift数据结构篇之 Queue
subtitle: Queue
date: 2018-05-04 13:31:53
categories: 数据结构与算法
---

队列，又称为伫列（queue），是先进先出（FIFO, First-In-First-Out）的线性表。在具体应用中通常用链表或者数组来实现。队列只允许在后端（称为rear）进行插入操作，在前端（称为front）进行删除操作。

<!-- more -->

队列的操作方式和堆栈类似，唯一的区别在于队列只允许新数据在后端进行添加。

### 在 Swift 中实现 Queue

与 [swift 数据结构之 Stack](http://struggleblog.com/2018/05/04/swift%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E7%AF%87%E4%B9%8BStack/) 相同, 也是采用 `struct` 来实现。

### 元素如何存储在栈中
入队中的元素如何存储呢？`Queue` 是我们自定义的一个结构体，需要一个 `backing storage` 来存储入栈的元素。这里我们使用 `Swift` 中的 `Array`来实现。

```
public struct Queue<T> {
    private var elements: [T] = []
}
```

由于内部储存不需要向外界暴露，所以使用了 `private`， 其内部存储的元素类型与入栈元素类型保持一致。

### 入队、出队

为 `Queue` 添加两个实例方法来达到入队、出队的目的。入队就是简单的向内部的数组追加元素，出队则是去除内部数组的第一个元素

```
public struct Queue<T> {
    ...
    
    // 入队
    public mutating func enqueue(_ element: T) {
        return elements.append(element)
    }
    // 出队
    public mutating func dequeue() -> T? {
        guard !elements.isEmpty else { return nil }
        return elements.removeFirst()
    }
}
```

### 取回队头元素

根据队列的特性，队头元素就是第一个入队的元素，也就是内部数组的第一个元素：

```
public struct Queue<T> {
    ...
    // 取回队头元素
    public func peek() -> T? {
        return elements.first
    }
}
```

### 队列元素个数、队列是否为空

```
public struct Queue<T> {
    
    ...
    
    public var count: Int {
        return elements.count
    }
    
    public var isEmpty: Bool {
        return elements.isEmpty
    }
}
```

### 清空队列

```
public struct Queue<T> {
    ...
    public mutating func clear() {
        elements.removeAll()
    }
}
```

### 初始化 Queue

```
public struct Queue<T> {
    ...
    // 创建一个空的队列
    public init() {} 
    // 从已有序列创建队列
    public init<S: Sequence>(_ s: S) where S.Iterator.Element == T {
        self.elements.append(contentsOf: s)
    }
}
```

### 使用数组字面量初始化

```
extension Queue: ExpressibleByArrayLiteral {
    public init(arrayLiteral elements: T...) {
        self.init(elements)
    }
}
```
需要将 `elements` 的类型由 `Self.ArrayLiteralElement`改为`Queue`中所用到的类型 `T`，在调用`public init<S: Sequence>(_ s: S)` 构造器来完成初始化。

```
let stack: Queue = [1, 2, 3, 4] // 需显示指明 Queue 的类型，如不指定则优先推断为 Array
```

### 实现 description 和debuDescription

```
extension Queue: CustomStringConvertible, CustomDebugStringConvertible {
    public var description: String {
        return elements.description
    }
    
    public var debugDescription: String {
        return elements.debugDescription
    }
}
```
### 遍历
```
extension Queue: Sequence {
    public func makeIterator() -> AnyIterator<T> {
        return AnyIterator(IndexingIterator(_elements: self.elements.lazy))
    }
}
```
### 通过 `subscript` 获取队列中的元素

要实现通过 `subscript` 获取队列中的元素，则需要遵从 `Collection`协议：

```
extension Queue: Collection {
    
    public func check(index: Int) {
        if index < 0 || index > count {
            fatalError("Index out of range")
        }
    }
    
    public var startIndex: Int {
        return 0
    }
    
    public var endIndex: Int {
        return count - 1
    }
    
    public func index(after i: Int) -> Int {
        return elements.index(after: i)
    }
    
    public subscript(index: Int) -> T {
        get {
            check(index: index)
            return elements[index]
        }
        set {
            check(index: index)
            elements[index] = newValue
        }
    }
}
```

### 完整代码

```
import Foundation

public struct Queue<T> {
    private var elements: [T] = []
    
    public init() {}
    
    public init<S: Sequence>(_ s: S) where S.Iterator.Element == T {
        self.elements.append(contentsOf: s)
    }
    
    public mutating func dequeue() -> T? {
        guard !elements.isEmpty else { return nil }
        return elements.removeFirst()
    }
    
    public mutating func enqueue(_ element: T) {
        return elements.append(element)
    }
    
    public func peek() -> T? {
        return elements.first
    }
    
    public mutating func clear() {
        elements.removeAll()
    }
    
    public var count: Int {
        return elements.count
    }
    
    public var isEmpty: Bool {
        return elements.isEmpty
    }
}

extension Queue: CustomStringConvertible, CustomDebugStringConvertible {
    public var description: String {
        return elements.description
    }
    
    public var debugDescription: String {
        return elements.debugDescription
    }
}

extension Queue: ExpressibleByArrayLiteral {
    public init(arrayLiteral elements: T...) {
        self.init(elements)
    }
}

extension Queue: Sequence {
    
    public func makeIterator() -> AnyIterator<T> {
        return AnyIterator(IndexingIterator(_elements: self.elements.lazy))
    }
}

extension Queue: Collection {
    public func check(index: Int) {
        if index < 0 || index > count {
            fatalError("Index out of range")
        }
    }

    public var startIndex: Int {
        return 0
    }

    public var endIndex: Int {
        return count - 1
    }

    public func index(after i: Int) -> Int {
        return elements.index(after: i)
    }

    public subscript(index: Int) -> T {
        check(index: index)
        return elements[index]
    }
}
```





