---
layout: post
published: false
title: Swift数据结构篇之 Stack
subtitle: Stack
date: 2018-05-04 10:40:49
categories: 数据结构与算法
---

栈（英语：stack）又称为栈或堆叠，是计算机科学中一种特殊的串列形式的抽象数据类型，其特殊之处在于只能允许在链表或数组的一端（称为堆栈顶端指针，英语：top）进行加入数据（英语：push）和输出数据（英语：pop）的运算。另外栈也可以用一维数组或链表的形式来完成。堆栈的另外一个相对的操作方式称为队列。 

<!-- more -->

由于堆栈数据结构只允许在一端进行操作，因而按照后进先出（LIFO, Last In First Out）的原理运作。  

堆栈数据结构使用两种基本操作：推入（压栈，push）和弹出（弹栈，pop）：
- 推入：将数据放入堆栈的顶端（数组形式或串列形式），堆栈顶端top指针加一
- 弹出：将顶端数据数据输出（回传），堆栈顶端数据减一。 

栈的基本特点：  
- 先入后出，后入先出。  
- 除头尾节点之外，每个元素有一个前驱，一个后继。

<html>
<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Data_stack.svg/391px-Data_stack.svg.png" />
<em>堆栈的简单示意图</em>
</p>
</html>

以上引用自[维基百科](https://zh.wikipedia.org/wiki/%E5%A0%86%E6%A0%88)

### 在 Swift 中实现 Stack

在 `Swift Standard Library` 中，几乎所有的数据结构都是用 `struct` 来实现的,为什么使用 `struct`, 可以参考 [Why Choose Struct Over Class?](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24232845?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)。在这里也使用 `struct` 来实现。  

由于栈中的元素可以是任意的，所以这里使用到了泛型。基本结构为：
```
public struct Stack<T> {
    ...
}
```

### 元素如何存储在栈中
压入栈中的元素如何存储呢？`Stack` 是我们自定义的一个结构体，需要一个 `backing storage` 来存储入栈的元素。这里我们使用 `Swift` 中的 `Array`来实现。

```
public struct Stack<T> {
    private var elements: [T] = []
}
```

由于内部储存不需要向外界暴露，所以使用了 `private`， 其内部存储的元素类型与入栈元素类型保持一致。

### 压栈、出栈

为 `Stack` 添加两个实例方法来达到压栈、出栈的目的。压栈就是简单的向内部的数组追加元素，出栈则是去除内部数组的最后一个元素

```
public struct Stack<T> {
    private var elements: [T] = []
    // 压栈
    public mutating func push(_ element: T) {
        elements.append(element)
    }
    
    // 出栈
    public mutating func pop() -> T? {
        return elements.popLast()
    }
}
```

### 取回栈顶元素

根据栈的特性，栈顶元素就是最后一个入栈的元素，也就是内部数组的最后一个元素：

```
public struct Stack<T> {
    private var elements: [T] = []
    
    public mutating func push(_ element: T) {
        elements.append(element)
    }
    
    public mutating func pop() -> T? {
        return elements.popLast()
    }
    // 取回栈顶元素
    public func peek() -> T? {
        return elements.last
    }
```

### 栈中元素个数，栈是否为空

```
public struct Stack<T> {
    private var elements: [T] = []
    
    public mutating func push(_ element: T) {
        elements.append(element)
    }
    
    public mutating func pop() -> T? {
        return elements.popLast()
    }
    
    public func peek() -> T? {
        return elements.last
    }
    // 栈是否为空
    public var isEmpty: Bool {
        return elements.isEmpty
    }
    // 栈中元素个数
    public var count: Int {
        return elements.count
    }
```

### 初始化 Stack

提供一个最基本的构造器
```
public init() {}
```
其实在 `swift` 中，编译器默认为 `结构体` 添加了一个包含包含所有成员的初始化构造器，所有这个我们提供的这个这也可以不写。

如果说我现在有一个序列(Sequence)，我想将其中的所有元素按顺序都放到栈中，怎么做呢，像下面这样？

```
let aSequence = [1, 2, 3, 4]
var stack = Stack<Int>()
for item in aSequence {
    stack.push(item)
}
```

这样看着很是不舒服，不够简洁。所以可以为`Stack`添加一个构造器，使其可以接收一个 `Sequence` 来完成初始化。

```
public init<S: Sequence>(_ s: S) where S.Iterator.Element == T {
    self.elements = Array(s)
}
```

这里要保证 `Sequence` 的迭代器的元素类型要和 `Stack` 中的元素类型要保持一致。

### 更方便的初始化

如果更加方便的来初始化我们的 `Stack` 呢？在使用 `swift standard library`提供的 `集合(set)` 的时候，可以像下面这样初始化：

```
let set: Set = [1, 2, 3]
```
等号后面是个 Array, 如何将给定的数组转为了集合呢？通过查看 `Set` 的定义的发现，其遵从了一个名为 `ExpressibleByArrayLiteral` 的协议，意思是`可用数组字面量表示`，需要实现一个初始化方法：

```
public init(arrayLiteral elements: Self.ArrayLiteralElement...)
```

那是不是我们的 `Stack` 遵从了该协议后就能使用数组字面量来表示了呢？ 答案是肯定的。

```
extension Stack: ExpressibleByArrayLiteral {

    public init(arrayLiteral elements: T...) {
        self.init(elements)
    }
}
```
需要将 `elements` 的类型由 `Self.ArrayLiteralElement`改为`Stack`中所用到的类型 `T`，在调用`public init<S: Sequence>(_ s: S)` 构造器来完成初始化。

```
let stack: Stack = [1, 2, 3, 4] // 需显示指明 stack 的类型，如不指定则优先推断为 Array
```

### debug

在 OC 中所有的类(除 `NSProxy` 外)都是 `NSObject` 的子类, 都遵从了 `NSObjectProtocol` 协议，所以可以实现一个`description` 和一个 `debugDescription` 属性，这两个属性非常有用。但我们的 `Stack` 并不是继承自 `NSObject`， 而是一个 `struct`, 如何实现这两个属性呢？比直接在 `Stack` 结构体中 直接声明这两个属性更加优雅的方式是：`extension` 我们的 `Stack`并遵从`CustomStringConvertible`, `CustomDebugStringConvertible` 这两个协议。

```
extension Stack: CustomStringConvertible, CustomDebugStringConvertible {
    public var description: String {
        return elements.description
    }
    
    public var debugDescription: String {
        return elements.debugDescription
    }
}
```

### 遍历

如何遍历我们的 `Stack` 实例呢？内部的 `Array` 对外不可见, 无法直接对其进行遍历。在 swift 标准库这提供了一个 `Sequence`协议，实现了该协议的实例就具有了可以被遍历的能力。

`Sequence` 协议非常简单，只需要实现一个方法就可以了。
```
public func makeIterator() -> Self.Iterator
```

该方法需要返回一个遵从 `IteratorProtocol` 协议的实例即可。

来看一下 `IteratorProtocol` 有哪些需要实现的方法或属性。遵从该协议只需要实现一个 `next` 方法，如下：

```
public mutating func next() -> Self.Element?
```

下面就来看下如何操作。

1. 定义一个遵从`IteratorProtocol`协议的结构体`ArrayIterator`, 并实现 `next`方法：  
    ```
    public struct ArrayIterator<T> : IteratorProtocol {
        
        init(elements: [T]){
            self.currentElement = elements
        }
        
        mutating public func next() -> T? {
            if self.currentElement.isEmpty { return nil }
            return self.currentElement.popLast()
        }
    }
    ```
    由于我们的堆栈实现使用一个数组作为其内部存储，因此我们将一个名为`currentElement`的实例变量定义为`[T]`，其中T是在采用`ArrayIterator`时定义的，因此它将保存传递给初始值设定项的数组。  
    然后我们实现`next`方法，该方法负责返回序列中的下一个元素。 如果我们已经到达数组的末尾，则返回`nil`。 这将向swift运行时发出信号，表示在`for ... in`循环中迭代它时没有更多元素可用。
2.  扩展 `Stack` 使其遵从 `Sequence` 协议，并实现 `makeIterator`:
    
    ```
    extension Stack: Sequence {
        public func makeIterator() -> ArrayIterator<T> {
            return ArrayIterator<T>(elements: self.elements)
        }
    }
    ```
    
### 代码优化
在上面的`遍历`中， 我们创建了一个遵从`IteratorProtocol`协议的结构体`ArrayIterator`, 来达到遍历的目的。在 swift 标准库中，已经完成了相应的工作，可以省去大量的样板代码。下面介绍三种新类型， 这些类型允许我们优化我们的 `Stack`，因此它也支持`lazy evaluation`:

- **AnyIterator<Element>** - An abstract `IteratorProtocol` base type. Use as a sequence type associated `IteratorProtocol` type.
- **AnySequence<Base>** - Creates a sequence that has the same elements as base. When used below the call to .lazy, it will return a `LazySequence` type
- **IndexingIterator<Elements>** - An iterator for an arbitrary collection. This is also the default iterator for any collection that doesn't declare its own.

所以遵从 `Sequence` 协议的 `extension` 就可以更新为：

```
extension Stack: Sequence {
    public func makeIterator() -> AnyIterator<T> {
        return AnyIterator(IndexingIterator(_elements: self.elements))
    }
}
```

### 完整代码

```
public struct Stack<T> {
    private var elements: [T] = []
    public init() {}
    
    public init<S: Sequence>(_ s: S) where S.Iterator.Element == T {
        self.elements = Array(s)
    }
    
    public mutating func pop() -> T? {
        return elements.popLast()
    }
    
    public mutating func push(_ element: T) {
        elements.append(element)
    }
    
    public func peek() -> T? {
        return elements.last
    }
    
    public var isEmpty: Bool {
        return elements.isEmpty
    }
    
    public var count: Int {
        return elements.count
    }
}

extension Stack: CustomStringConvertible, CustomDebugStringConvertible {
    public var description: String {
        return elements.description
    }
    
    public var debugDescription: String {
        return elements.debugDescription
    }
    
}

extension Stack: ExpressibleByArrayLiteral {

    public init(arrayLiteral elements: T...) {
        self.init(elements)
    }
}

extension Stack: Sequence {
    public func makeIterator() -> AnyIterator<T> {
        return AnyIterator(IndexingIterator(_elements: self.elements.lazy))
    }
}

```















