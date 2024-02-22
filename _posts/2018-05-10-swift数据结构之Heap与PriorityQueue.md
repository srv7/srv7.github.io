---
layout: post
published: false
title: Swift数据结构之 Heap与PriorityQueue
subtitle: Heap&&PriorityQueue
date: 2018-05-10 16:12:46
categories: 数据结构与算法
---

## Heap
堆数据结构在1964年由J.W.J.Williams首次引入，作为堆排序算法的数据结构。

理论上，堆类似于二叉树数据结构（类似于二叉搜索树）。 堆是一棵树，并且树中的所有节点都有0,1或2个子节点。如下图所示：

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr55t9djwej20b407s3yf.jpg"/>
</p>
</html>

<!-- more -->

堆中的元素部分按其优先级排序。 树中的每个节点比其子节点具有更高的优先级。 有两种不同的表示方式可以表示值的优先级：
- **maxheaps**：具有较高值的元素代表较高的优先级。 
- **minheaps**：具有较低值的元素代表较高的优先级。

堆也具有紧凑的高度。 如果你认为堆有层次，就像这样：

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr55z6slsrj20fp07t0ss.jpg"/>
</p>
</html>

- 堆总是趋向于使用最小的层数来包含所有的元素，在新起一行之前必须保证已有`行`都已被填充。  
- 每当我们向堆中添加节点时，我们都会将它们添加到不完整级别的最左边的位置。  
- 每当我们从堆中移除节点时，我们从最底层移除最右边的节点。看图：

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr56dvxav3j20gp0c9wfl.jpg"/>
</p>
</html>


### 删除优先级最高的元素

`堆`作为`优先级队列`很有用，因为根节点包含堆中具有最高优先级的元素。然而，简单地删除根节点后留下的不是堆。或者说是留下了两个堆！

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr56mokwo2j20gp0a1jrc.jpg"/>
</p>
</html>

相反地，将根节点与堆中的最后一个节点进行交换。然后删除它：

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr56ohwu9dj20gp0a174b.jpg"/>
</p>
</html>

然后，将新的根节点与每个子节点进行比较，并将其与具有最高优先级的任何子节点进行交换。

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr56psd3j8j20gp07tq2w.jpg"/>
</p>
</html>

现在，新的根节点是优先级最高的元素，但堆可能还未完成排序，再次比较新的子节点和子节点，并将其与具有最高优先级的子节点进行交换。

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr56vjbtjxj20gp07tmx4.jpg"/>
</p>
</html>

继续筛选，直到前一个元素的优先级高于其子元素，或者它成为叶子节点。

### 添加新元素

添加新元素使用了非常相似的技术。首先，在堆的不完整`行`的最左边位置添加新元素：

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr576goff2j20gp0a1aa1.jpg"/>
</p>
</html>


然后比较新元素的优先级与其父元素，如果它具有更高的优先级，我们将筛选出来。
<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr5771o62dj20gp0a1t8p.jpg"/>
</p>
</html>

继续筛选，直到新元素的优先级低于其父元素，或者它成为堆的根。
<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr577kspx9j20gp0a10sq.jpg"/>
</p>
</html>

### Practical Representation 
如果您了解二进制搜索树，那么您可能会疑惑，堆数据结构没有Node数据类型来包含其元素和指向其子元素的链接。实际上堆数据结构是一个数组！

堆中的每个节点都分配了一个索引。我们首先将0分配给根节点，然后我们遍历层次，从左到右统计每个节点：

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr57gfcu0hj20gp0a1dfu.jpg"/>
</p>
</html>

然后，如果我们使用这些索引来创建一个数组，每个元素都存储在索引位置，它将如下所示：

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr57hatbkkj20gp06pweg.jpg"/>
</p>
</html>

可以通过一些公式将每个节点连接到它的子节点，值得注意的是每个`level`节点数量是其上一级的两倍。

给定索引处的节点`i`，其左侧子节点可以在`2i + 1`处找到，并且其右侧子节点可以在`2i + 2`处找到。

<html>
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fr57kos6xwj20k10ehjrt.jpg"/>
</p>
</html>

这就是为什么堆是一个紧凑树以及为什么我们将每个新元素添加到最左边位置的原因：我们实际上是将新元素添加到数组中，并且我们不能留下任何空白。

> 注意：这个数组没有排序。正如你从上图中可能已经注意到的那样，堆关心的节点之间的唯一关系是父母比他们的孩子有更高的优先级。堆不关心哪个左边的孩子和右边的孩子具有更高的优先级。靠近根节点的节点并不总是比距离更远的节点更高的优先级。

### swift 中的实现

```
public struct Heap<Element> {
    /// 内部存储
    private var elements: [Element]
    /// 决定优先级的闭包
    private let priorityFunction: (Element, Element) -> Bool
    //// 堆是否为空
    public var isEmpty: Bool {
        return elements.isEmpty
    }
    /// 堆数量
    public var count: Int {
        return elements.count
    }
    
    /// 取回优先级最高的元素，不移除元素
    public func peek() -> Element? {
        return elements.first
    }
    
    /// 给定索引是否是根节点
    public func isRoot(_ index: Int) -> Bool {
        return index == 0
    }
    
    /// 给定节点的左侧子节点索引（如果有的话）
    /// 左侧节点的所有总是奇数
    @inline(__always)
    public func leftChildIndex(of index: Int) -> Int {
        return index * 2 + 1
    }
    
    /// 给定节点的右侧子节点索引（如果有的话）
    /// 右侧节点的所有总是偶数
    @inline(__always)
    public func rightChlidIndex(of index: Int) -> Int {
        return index * 2 + 2
    }
    
    /// 给定节点的父节点索引
    @inline(__always)
    public func parentIndex(of index: Int) -> Int {
        return (index - 1) / 2
    }
    
    /// firstIndex 索引对应的节点的优先级是否大于 secondIndex 索引对应的节点的优先级
    public func isHighPriority(at firstIndex: Int, than secondIndex: Int) -> Bool {
        return priorityFunction(elements[firstIndex], elements[secondIndex])
    }
    
    /// 比较父节点与子节点的优先级 返回优先级较大的节点的索引
    /// 如果子节点索引大于数组大小 则直接返回父节点索引
    /// 插入新元素时会用到此方法，比较它与父节点的优先级
    public func highestPriorityIndex(of parentIndex: Int, and childIndex: Int) -> Int {
        if childIndex < count, isHighPriority(at: childIndex, than: parentIndex) {
            return childIndex
        }
        return parentIndex
    }
    
    /// 返回给定节点的子节点中优先级最高的子节点序列
    /// 父节点先与左侧子节点比较，得出高优先级的索引，再把得到的索引与右侧子节点比较，得出最高优先级的子节点（结果可能是父节点优先级最大）
    /// 删除元素后，会用此方法重新排序
    public func highestPriorityIndex(for parent: Int) -> Int {
        return highestPriorityIndex(of: highestPriorityIndex(of: parent, and: leftChildIndex(of: parent)), and: rightChlidIndex(of: parent))
    }
    
    /// 简单封装交换数组中两个元素的方法
    private mutating func swapElementsAt(_ firstIndex: Int, _ secondInx: Int) {
        guard firstIndex != secondInx else {
            return
        }
        elements.swapAt(firstIndex, secondInx)
    }
    
    /// 添加新元素
    /// 将元素追加到数组最后（即 添加到不完整级别的最左边的位置）
    /// 然后将其父节点比较优先级（swim：向上浮）
    public mutating func insert(_ element: Element) {
        elements.append(element)
        swim(elementAtIndex: count - 1)
    }
    
    public mutating func insert<S: Sequence>(_ sequence: S) where S.Iterator.Element == Element {
        for item in sequence {
            insert(item)
        }
    }
    
    /// 将给定索引位置的节点向上浮动
    private mutating func swim(elementAtIndex index: Int) {
        /// 父节点索引
        let parent = parentIndex(of: index)
        
        /// 如果给定索引是根节点 或者其优先级不高于其父节点 则直接返回，停止浮动
        guard !isRoot(index), isHighPriority(at: index, than: parent) else {
            return
        }
        /// 否则，就交换两处索引的节点
        swapElementsAt(index, parent)
        /// 交换后，其现在索引是交换前父节点的索引 也就是 第一行的`parent`，进行递归，直到 （变为根节点 || 不大于其父节点优先级）
        swim(elementAtIndex: parent)
    }
    
    /// 删除优先级最高的元素， 也就是根节点元素
    @discardableResult
    public mutating func remove() -> Element? {
        /// 如果堆为空，则直接返回 nil
        guard !isEmpty else { return nil }
        /// 根据上面说的，删除一个元素，要先其与堆中最后一个元素互换位置后删除， 然后将`最后一个元素`向下沉
        swapElementsAt(0, count - 1)
        let element = elements.removeLast()
        if !isEmpty {
            sink(elementAtIndex: 0)
        }
        return element
    }
    
    /// 将元素下沉
    /// 自身与两个子节点进行优先级比较，如果优先级比两个子节点高则不再进行操作
    /// 将自己与优先级最高的子节点交换位置，
    /// 交换位置后，进行递归操作
    private mutating func sink(elementAtIndex index: Int) {
        let childIndex = highestPriorityIndex(for: index)
        if index == childIndex { return }
        swapElementsAt(index, childIndex)
        sink(elementAtIndex: childIndex)
    }
    
    public init(elements: [Element] = [], sortedBy: @escaping (Element, Element) -> Bool) {
        self.elements = elements
        self.priorityFunction = sortedBy
        buildHeap()
    }
    
    private mutating func buildHeap() {
        for index in (0 ..< count / 2).reversed() {
            sink(elementAtIndex: index)
        }
    }
    
    /// 将指定位置的元素移除
    /// 如果指定位置超出内部数组大小 直接 return nil
    /// 如果指定位置是内部数组最后一个元素 将其移除出数组并返回
    /// 其他情况：将其与最后一个元素互换位置，操作基本上 remove 一致
    @discardableResult
    public mutating func remove(at index: Int) -> Element? {
        guard index < elements.count else { return nil }
        
        let size = count - 1
        
        if index == size {
            return elements.removeLast()
        }
        swapElementsAt(index, size)
        let element = elements.removeLast()
        sink(elementAtIndex: index)
        return element
    }
    
    /// 移除指定位置的元素然后添加新的元素
    public mutating func replace(index: Int, value: Element) {
        guard index < count else { return }
        remove(at: index)
        insert(value)
    }
}

extension Heap: CustomDebugStringConvertible, CustomStringConvertible {
    public var description: String {
        return elements.description
    }
    
    public var debugDescription: String {
        return elements.debugDescription
    }
}

public extension Heap where Element: Equatable {
    /// 元素的索引
    public func index(of element: Element) -> Int? {
        return elements.index(where: { $0 == element })
    }
    
    /// 移除指定元素
    @discardableResult
    public mutating func remove(_ element: Element) -> Element? {
        if let index = index(of: element) {
            return remove(at: index)
        }
        return nil
    }
}


```

## Priority Queue

优先队列也是一个队列，满足 `FIFO`, 最重要的最高的元素总是在队头。
队列可以是最大优先级队列（最大元素第一）或最小优先级队列（最小元素第一）。

### 为什么使用优先队列？
优先级队列对于需要处理大量项目的算法以及需要重复确定哪个算法现在最大或最小的算法很有用。

Examples:
- 模拟事件驱动。每个事件都有一个时间戳，并且希望按照时间戳的顺序执行事件。优先级队列可以很容易地找到需要模拟的下一个事件。
- Dijkstra的图搜索算法使用优先级队列来达到最小代价。
- 霍夫曼编码数据压缩。该算法构建压缩树。它一再需要找到具有最小频率的两个没有父节点的节点。
- Lots of other places!


### 实现
有了前面的 `Heap`, 就可以很快地实现 `Priority Queue`了


```
public struct PriorityQueue<T> {
    private var heap: Heap<T>
    
    public init(sortBy: @escaping (T, T) -> Bool) {
        heap = Heap(sortedBy: sortBy)
    }
    
    public var isEmpty: Bool {
        return heap.isEmpty
    }
    
    public var count: Int {
        return heap.count
    }
    
    public func peek() -> T? {
        return heap.peek()
    }
    
    public mutating func enqueue(_ element: T) {
        return heap.insert(element)
    }
    
    @discardableResult
    public mutating func dequeue() -> T? {
        return heap.remove()
    }
    
    public mutating func changePriority(index i: Int, value: T) {
        heap.replace(index: i, value: value)
    }
}

extension PriorityQueue: CustomStringConvertible, CustomDebugStringConvertible {
    public var description: String {
        return heap.description
    }

    public var debugDescription: String {
        return heap.debugDescription
    }
}
```


## Reference

 - [Swift Algorithm Club: Heap and Priority Queue Data Structure](https://www.raywenderlich.com/160631/swift-algorithm-club-heap-and-priority-queue-data-structure)
 - [swift-algorithm-club](https://github.com/raywenderlich/swift-algorithm-club)
 - [Swift Data Structure and Algorithms - Erik Azar](https://www.google.com/search?q=Swift+Data+Structure+and+Algorithms+-+Erik+Azar&rlz=1C5CHFA_enUS707US720&oq=Swift+Data+Structure+and+Algorithms+-+Erik+Azar&aqs=chrome..69i57.546j0j7&sourceid=chrome&ie=UTF-8)
 - [Heaps and Priority Queues](https://www.hackerearth.com/zh/practice/notes/heaps-and-priority-queues/)