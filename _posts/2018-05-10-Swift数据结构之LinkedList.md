---
layout: post
published: false
title: Swift 数据结构之 LinkedList
subtitle: LinkedList
date: 2018-05-10 19:26:13
categories: 数据结构与算法
---

链表（`Linked list`）是一种常见的基础数据结构，是一种线性表，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的指针。由于不必须按顺序存储，链表在插入的时候可以达到`O(1)`的复杂度，比另一种线性表顺序表快得多，但是查找一个节点或者访问特定编号的节点则需要`O(n)`的时间，而顺序表相应的时间复杂度分别是`O(logn)`和`O(1)`。

<!-- more -->

## 优缺点
使用链表结构可以克服数组链表需要预先知道数据大小的缺点，链表结构可以充分利用计算机内存空间，实现灵活的内存动态管理。

但是链表失去了数组随机读取的优点，同时链表由于增加了结点的指针域，空间开销比较大。

在计算机科学中，链表作为一种基础的数据结构可以用来生成其它类型的数据结构。链表通常由一连串节点组成，每个节点包含任意的实例数据（`data fields`）和一或两个用来指向上一个/或下一个节点的位置的链接（`links`）。链表最明显的好处就是，常规数组排列关联项目的方式可能不同于这些数据项目在记忆体或磁盘上顺序，数据的访问往往要在不同的排列顺序中转换。而链表是一种自我指示数据类型，因为它包含指向另一个相同类型的数据的指针（链接）。链表允许插入和移除表上任意位置上的节点，但是不允许随机存取。

## 分类
链表有很多种不同的类型：单向链表，双向链表以及循环链表。

- **单向链表**: 单向链接列表是链接列表，其中每个节点仅具有对下一个节点的引用

- **双向链表**: 其中每个节点具有对前一个节点和下一个节点的引用。
    需要跟踪列表开始和结束的位置。通常通过称为头部(`head`)和尾部(`tail`)的指针来完成。

## 实现

通过上面的介绍，我们了解到链表其实就是一个个节点（`node`）所串起来的。现在就来实现这个`node`.

### 实现 Node

```
fileprivate class Node<T> {
    /// 指向下一节点的指针
    fileprivate var next: Node<T>?
    
    /// 节点中存放的数据
    fileprivate var data: T
    
    /// 初始化一个 node， 保存数据， 下一节点指向 空
    init(data: T) {
        self.data = data
        next = nil
    }
}

```

值得注意的是：这里 `Node` 是使用 `class` 来实现的，而不是 `struct`，因为 `struct` 是值类型，传递值类型时，接收者得到的是他的一个拷贝，而不是其本身。而在传递引用类型时，接受者得到的是对它的一个引用。所以这里需要使用 `class` 来实现。

### 实现链表（以单向链表为例）

希望实现的链表具有以下功能:
- push - 添加一个节点
- pop - 删除头节点
- peek - 查看头节点
- isEmpty - 是否为空
- count - 节点数量

```
public struct LinkedList<Element> {
    
    fileprivate class Node<T> {
        /// 指向下一节点的指针
        fileprivate var next: Node<T>?
        
        /// 节点中存放的数据
        fileprivate var data: T
        
        /// 初始化一个 node， 保存数据， 下一节点指向 空
        init(data: T) {
            self.data = data
            next = nil
        }
    }
    
    /// 头节点指针
    fileprivate var head: Node<Element>? = nil
    /// 节点个数计数器
    private var _count: Int = 0
    
    /// 创建一个空链表
    public init() {}
    
    /// 从已有序列创建一个链表
    public init<S: Sequence>(_ sequence: S) where S.Iterator.Element == Element {
        self.init()
        /// 将元素逐个加入链表
        sequence.forEach { push($0) }
    }
    
    public var isEmpty: Bool {
        return _count == 0
    }
    
    public var count: Int {
        return _count
    }
    
    public func peek() -> Element? {
        return head?.data
    }
    
    /// 添加一个元素到链表
    public mutating func push(_ element: Element) {
        /// 创建一个 node
        let node = Node(data: element)
        push(node)
    }
    
    private mutating func push(_ node: Node<Element>) {
        /// 将其 next 指向当前 头节点（head）(如当前没有头节点则 next 指向空， 这个节点是链表的第一个节点)
        node.next = head
        /// 然后将头节点指针只想这个节点
        head = node
        /// 计数器加1
        _count += 1
        
    }
    
    /// 删除头节点
    public mutating func pop() -> Element? {
        /// 如果链表为空，则直接返回 nil
        if isEmpty {
            return nil
        }
        /// 得到头节点数据
        let item = head?.data
        /// 将头节点 指向下一个节点
        head = head?.next
        /// 计数器减1
        _count -= 1
        /// 返回更改前的头节点数据
        return item
    }
    
    /// 通过下标获取数据
    public subscript(index: Int) -> Element {
        let node = self.node(at: index)
        return node.data
    }
    
    /// 找出对应索引的节点
    private func node(at index: Int) -> Node<Element> {
        assert(head != nil, "Empty List")
        assert(index >= 0, "index must be greater than 0")
        
        if index == 0 {
            return head!
        } else {
            var node = head?.next
            for _ in 1 ..< index {
                node = node?.next
                if node == nil {
                    break
                }
            }
            assert(node != nil, "index is out of bounds.")
            return node!
        }
    }
    
    /// 在指定索引出插入数据
    public mutating func insert(_ element: Element, at index: Int) {
        let node = Node(data: element)
        insertNode(node, at: index)
    }
    /// 在指定索引出插入节点
    private mutating func insertNode(_ node: Node<Element>, at index: Int) {
        var index = index
        if index > count - 1 {
            index = count - 1
        }
        if index == 0 {
            push(node)
        } else {
            let previous = self.node(at: index - 1)
            let next = previous.next
            node.next = next
            previous.next = node
        }
    }
    
    /// 清空链表
    public mutating func clear() {
        head = nil
    }
    /// 移除指定位置的元素
    @discardableResult
    public mutating func remove(at index: Int) -> Element? {
        assert(index <= count - 1, "index is out of bounds.")
        if index == 0 {
            return pop()
        } else if index == count - 1 {
            let previous = self.node(at: index - 1)
            let deleteNode = previous.next
            previous.next = nil
            return deleteNode?.data
        } else {
            let previous = self.node(at: index - 1)
            let deleteNode = previous.next
            let next = deleteNode?.next
            previous.next = next
            return deleteNode?.data
        }
    }
}
```

### 从数组串字面量创建

```
/// 支持使用数组字面量创建
extension LinkedList: ExpressibleByArrayLiteral {
    public init(arrayLiteral elements: Element...) {
        self.init(elements)
    }
}
```

### 支持 for in 
只需要遵从 `Sequence` 协议并实现协议方法即可
```
/// 支持 `for in`
extension LinkedList: Sequence {
    
    public struct Iterator: IteratorProtocol {
        private var head: Node<Element>?
        
        fileprivate init(head: Node<Element>?) {
            self.head = head
        }
        
        public mutating func next() -> Element? {
            guard head != nil else { return nil }
            let item = head?.data
            head = head?.next
            return item
        }
    }
    public func makeIterator() -> LinkedList<Element>.Iterator {
        return Iterator(head: head)
    }
}
```

### description，debugDescription

```
extension LinkedList: CustomStringConvertible, CustomDebugStringConvertible {
    public var description: String {
        var d = "["
        var lastNode = head
        while lastNode != nil  {
            d = d + "\(lastNode!.data)"
            lastNode = lastNode?.next
            if lastNode != nil {
                d += ","
            }
        }
        d += "]"
        return d
        
    }
    
    public var debugDescription: String {
        var d = "["
        var lastNode = head
        while lastNode != nil  {
            d = d + "\(lastNode!.data)"
            lastNode = lastNode?.next
            if lastNode != nil {
                d += ","
            }
        }
        d += "]"
        return d
    }
}
```

## 应用（实现 `Stack`）

```
public struct Stack<Element> {
    private var elements: LinkedList<Element> = LinkedList()
    
    public var isEmpty: Bool {
        return elements.isEmpty
    }
    
    public var count: Int {
        return elements.count
    }
    
    public mutating func push(_ element: Element) {
        elements.push(element)
    }
    
    public mutating func pop() -> Element? {
        return elements.pop()
    }
    
    public func peek() -> Element? {
        return elements.peek()
    }
}
```

# Reference

- [wikipedia](https://zh.wikipedia.org/wiki/%E9%93%BE%E8%A1%A8)
- [Swift Data Structure and Algorithms - Erik Azar](https://www.google.com/search?q=Swift+Data+Structure+and+Algorithms+-+Erik+Azar.pdf&rlz=1C5CHFA_enUS707US720&oq=Swift+Data+Structure+and+Algorithms+-+Erik+Azar.pdf&aqs=chrome..69i57.7466j0j7&sourceid=chrome&ie=UTF-8)
- [swift-algorithm-club](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Linked%20List/LinkedList.playground/Contents.swift)
- [Swift Algorithm Club: Swift Linked List Data Structure](https://www.raywenderlich.com/144083/swift-algorithm-club-swift-linked-list-data-structure)