---
layou: post
title: Swift数据结构篇之 CircularBuffer
subtitle: CircularBuffer
date: 2018-05-08 15:16:56
categories: 数据结构与算法
published: false
---

环形队列是一个首尾相连的`FIFO`（命名管道）的数据结构，它采用数组的线性空间,  是一种用于表示一个固定尺寸、头尾相连的缓冲区的数据结构，适合缓存数据流。

<html>
<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/Circular_buffer.svg/200px-Circular_buffer.svg.png" />
</p>
<p align="center">
<em>圆形缓冲区的概念图示</em>
</p>
</html>

<!-- more -->

### 原理
内存上没有环形结构，因此环形队列利用数组的线性空间来实现。当数据到了尾部时，它将转回到`0`位置来处理。这个转回操作通过取模来执行。

### 构造
逻辑上，将数组`q[0]`与`q[MAXN-1]`相连接，形成一个存放队列的环形空间。用数组下标来标明队列的读、写位置。`head` 指向可以读的位置，`tail` 指向可以写的位置。(`head` 队列中的第一个元素位置，`tail` 队列中的最后一个元素位置的下一个位置)

环形队列的关键是判断队列为空或者满, 例如初始时，队列中没有元素，`head` 和 `tail` 均为 `0`,可以说是 `head` 追上 `tail`，当队列元素已满的时候，`tail` 指向队列头，可以说是`tail`追上`head`。 于是可以说：
`head` 追上 `tail` 则为`空`, `tail`追上`head`则为`满`。

### 实现

这里使用数组来实现 CircularBuffer。实现步骤与前面的 `Queue` 文章相同, 这里直接给出代码，代码里会给出必要的说明。另外，可以指定队列的覆盖模式，当队列已满的时候，在进行入队时采取何种措施来进行入队，忽略或者覆盖已有元素。


```
import Foundation
/// 队列默认容量大小
fileprivate struct Constant {
    fileprivate static let defaultQueueCapacity: Int = 16
}

public struct CircularBuffer<T> {
    /// 元素覆盖操作
    public enum OverwriteOperation {
        case ignore /// 忽略新元素
        case overwrite /// 覆盖 head
    }
    /// 内部存储
    private var elements: [T?]
    /// 可读取的位置
    private var head: Int = 0
    /// 可写入的位置
    private var tail: Int = 0
    /// 队列容量， 外部只能读取
    private(set) var capacity: Int
    /// 元素计数器
    private var internalCount: Int = 0
    /// 元素覆盖操作
    private(set) var overwriteOperation: OverwriteOperation = .overwrite
    
    /// 队列元素个数，直接返回内部的 元素计数器
    public var count: Int {
        return internalCount
    }
    
    /// 队列是否为空，如果 元素个数为0 则为空
    public var isEmpty: Bool {
        return internalCount == 0
    }
    
    /// 队列是否为满，如果元素个数与容量相等，则为满
    public var isFull: Bool {
        return internalCount == capacity
    }
    
    /// designed initializer
    init() {
        elements = [T?]()
        capacity = Constant.defaultQueueCapacity
    }
    
    init(_ size: Int, overwriteOperation: OverwriteOperation = .overwrite) {
        elements = [T?]()
        capacity = size
        self.overwriteOperation = overwriteOperation
    }
    
    /// 从已有序列创建
    init<S: Sequence>(_ elements: S, capacity: Int) where S.Iterator.Element == T {
        self.init(capacity)
        /// 将序列中的元素逐个入栈，涉及到对 head 、tail的处理，不能直接放入内部存储的数组中
        elements.forEach { enQueue($0) }
    }
    
    /// 入队
    public mutating func enQueue(_ element: T) {
        /// 如果队列已满，则看覆盖操作选项，如果忽略新值则直接 return，若是覆盖元素则将队列头元素做出队操作
        if isFull {
            switch overwriteOperation {
            case .ignore: return
            case .overwrite: deQueue()
            }
        }
        /// 如果内部存储数组的元素个数大于等于队列容量，则直接将对应 index 位置的元素设置为新值，否则直接追加在数字尾部
        if elements.count >= capacity {
            elements[tail] = element
        } else {
            elements.append(element)
        }
        /// 更新尾部指针
        tail = incrementPointer(tail)
        /// 元素计数器加 1，即元素个数加 1
        internalCount += 1
    }
    
    /// 出队, 返回出队元素
    @discardableResult
    public mutating func deQueue() -> T? {
        /// 如果队列为空，直接返回 nil
        if isEmpty {
            return nil
        }
        /// 取出队头元素
        let element = elements[head]
        /// 内部数组相应位置值为 nil
        elements[head] = nil
        /// 更改 head 指针
        head = incrementPointer(head)
        /// 队列元素个数减 1
        internalCount -= 1
        /// 返回
        return element
    }
    
    /// 查看队头元素
    public func peek() -> T? {
        return elements[head]
    }
    
    /// 清空队列
    public mutating func clear() {
        head = 0
        tail = 0
        internalCount = 0
        elements.removeAll()
    }
    
    /// 通过模运算更改指针 important！，实现环形队列的基础
    private func incrementPointer(_ pointer: Int) -> Int {
        let x = pointer + 1
        return x % capacity
        /// 也可以通过位与运算来实现
        // return (pointer + 1) & (capacity - 1)
    }
}

/// description
extension CircularBuffer: CustomStringConvertible, CustomDebugStringConvertible {
    public var description: String {
        return elements.description
    }

    public var debugDescription: String {
        return elements.debugDescription
    }
}
/// 通过字面量表示
extension CircularBuffer: ExpressibleByArrayLiteral {
    public init(arrayLiteral elements: T...) {
        self.init(elements, capacity: elements.count)
    }
}

/// 支持遍历
extension CircularBuffer: Sequence {
    public func makeIterator() -> AnyIterator<T?> {
        /// 如果队列为空，则直接返回空的数组进行迭代
        guard count > 0 else {
            return AnyIterator(IndexingIterator(_elements: [].lazy))
        }
        
        var data = [T?]()
        /// 逻辑如下图
        if head >= tail {
            data.append(contentsOf: elements[head ..< capacity])
            let end = tail - 0
            data.append(contentsOf: elements[0 ..< end])
        } else {
            data.append(contentsOf: elements[head ..< tail])
        }
        
        return AnyIterator(IndexingIterator(_elements: data.lazy))
    }
}

```

![](https://ws1.sinaimg.cn/large/b92f96b9ly1fr3yc2zwmoj20u01hcqky.jpg)
