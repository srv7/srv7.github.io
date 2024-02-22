---
layout: post
title: RxSwift中的内存管理—DisposeBag
date: 2020-08-06 00:05:53
subtitle: memory-managment-rxswift
categories: Rx
---

原文：[Memory management in RxSwift – DisposeBag](http://adamborek.com/memory-managment-rxswift/)

许多 RxSwift 初学者都有 `DisposeBag` 方面的疑问。DisposeBag 并不是 iOS 开发中的标准， 也不是其他 Rx 实现的标准。简单来说，它是 RxSwift 在 iOS 平台上进行内存管理的方式。

在本文中，我将回答几个问题， 例如什么是 DisposeBag、Disposble。笼统地谈谈 RxSwift 的 ARC 内存管理。最后谈谈如何在使用 RxSwift 的时候如何避免内存泄漏。希望你会喜欢。📚💪

<!--more-->

<img src="http://i2.wp.com/adamborek.com/wp-content/uploads/2017/06/pexels-photo-131050-1.jpg?w=1080"/>


## Observable && 内存管理

当实现一个用于处理异步事件的库时，由于 iOS 引用计数的存在，你需要了解一些事情。

描述问题的最简单方法是通过一个例子来描述它：

### 取消还是不取消? 🤔

假设有一个 `Observable` , 代表一个 REST API 的调用。当对其调用 `subscribe`时，它会将请求发送到服务器，然后等待响应。假设是在 `UIViewController` 的 `viewDidLoad` 中调用了 `subscribe`。

这是一个很简单的例子，但你需要注意一点，用户可能会在任何时候点击导航的返回按钮。在正确管理内存的情况下，返回前一个页面，当前的 `UIViewController` 会被 **deallocate**，并取消 `Observable`，因为他失去了对 UIViewController 的引用。

因此，请求将没有机会完成。

有时这是一种预期的行为，但有时候却希望在收到响应后，尽管页面已经销毁，开发人员可以控制 Observable 何时被终止。

### 内存资源是有限的

另一件关于内存管理的事情是内存是 **内存资源是有限的**。

`Observables` 可以在其实现中存储一些变量，也可以存储你传递给他们的内容。

这意味着 `Observables` 可能会为满足其内部需求而分配一些内存空间。

另一方面，你可能知道，`Observable` 的特质之一是在其接收到 **completed**和 **error** 事件后，不会再继续发送事件。

如果它不再发送新事件，那么保留其内部资源又有什么意义呢？一个好主意是清理并释放 `Observable` 保留的内存。

为了能够清理 `Observables`，我们需要有按需清理 `Observables` 的机会。这也就是为什么 `subscribe` 方法会返回 `Disposable`。

> 引用计数是一种内存管理方式，在 iOS 平台上，每个对象都有一个 retainCount 属性。当一个对象被强引用时，其 retainCount 会加1，当强引用消除时，retainCount 减1，当 retainCount 为 0 时，改对象会被销毁。



## Disposable — 故事的起源

`Disposable` 只是一个协议，包含了一个 `dispose` 方法:

```swift
public protocol Disposable {
    func dispose()
}
```

当你 `subscribe` 一个 `Observable` 时，`Disposable`  强引用了  `Observable` ，同时 `Observable` 也保留了对 `Disposable` 强的引用。（Rx 创建了一种循环引用）。因此，如果用户退出当前页面， `Observable` 将不会被释放，除非你希望如此。

为了解除循环引用，需要有人对 Observable 来调用 dispose。如果 `Observable` 自行结束（ 发送了 completed 或 error 事件）,将自动打破循环。其他情况下，调用 dispose 的责任将落在我们手中。

最简单的方式是在 deinit 方法中调用 dispose:

```swift
final class MyViewController: UIViewController {
    var subscription: Disposable?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        subscription = theObservable().subscribe(onNext: {
            // handle your subscription
        })
    }
    
    deinit {
        subscription?.dispose()
    }
}	
```

这个解决方案很简单，但它不可扩展。想象一下，你的类需要有多少这种字段。

改进一下，你可以定义一个 disposable 的数组` [Disposable]`，遍历数组对每一个 Disposable 调用 dispose:

```swift
final class MyViewController: UIViewController {
    var subscriptions = [Disposable]()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        subscriptions.append(theObservable().subscribe(onNext: {
            // handle your subscription
        }))
    }
    
    deinit {
        subscriptions.forEach { $0.dispose() }
    }
}
```

这样看起来好多了，并且可扩展了。无论有多少个订阅，在 deinit 时 看起来都是一样的。

然而，这并不是改进的尽头。当前这种方式强制你在 deinit 的时候手动处理 disposable。

是的，你可能已经想到了。可以使用 `DisposeBag` 代替 `[DisposeBag]`

```swift
final class MyViewController: UIViewController {
    let disposeBag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()
        theObservable().subscribe(onNext: {
                // handle your subscription
        })
        .disposed(by: disposeBag)
    }
}
```

![](http://i2.wp.com/media.giphy.com/media/l0HlFZfztLH0oGtgY/giphy.gif?zoom=2&w=1080&ssl=1)

等一下，deinit 去哪了？

`DisposeBag` 酷的地方在于它**负责**对在它里面的每个 `Disposable` 调用 `dispose`。

那何时中调用` dispose` 呢？`DisposeBag` 在 `deinit` 的时候对包含在它里面的 `Disposable` 调用 `dispose`。也就是说，当 `DisposeBag` 失去 `UIViewController` 的引用时，其 retainCount 为 0，所以它会被释放，此时它将对所有的 `Disposable` 调用 `dispose`。

![](http://i2.wp.com/media.giphy.com/media/vEgtLzJo8n7qg/giphy.gif?zoom=2&w=1080&ssl=1)





## DisposeBag && 循环引用 😱

在 `deinit ` 里调用 `dispose` 似乎是最简单的清理内存的方式。但它只能在 `deinit` 被调用时才成立。

使用 `DisposeBag`，很容易在 `DisposeBag` 和 `UIViewController `之间产生循环引用。`DisposeBag` 将一直等待被释放，而不会对其管理的 `Disposables`  调用  `dispose`。

你需要记住的是，每个操作符都默认地对其捕获的变量进行强引用。

```swift
let parsedObject = theObservable
    .map { json in
        return self.parser.parse(json)
    }
```

在上面的例子中，`transformedObservable` 强引用了 `self`，因为在 map 的闭包中使用了 `self`，这种行为是 swift 使用引用计数的方式， 以确保一切都在需要时分配。

上面的代码并没有产生循环引用。不幸的是，稍加改动，就形成了循环引用问题。

```swift
final class MyViewController: UIViewController {
    private let disposeBag = DisposeBag()
    private let parser = MyModelParser()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        let parsedObject = theObservable
            .map { json in
                return self.parser.parse(json)
            }
            
        parsedObject.subscribe(onNext:{ _ in 
            //do something
        })
        .disposed(by: disposeBag)
    }
}
```

引起循环引用的代码是 `.disposed(by: disposeBag)` 和  `map操作符。

因为将 **Disposable** 加入到 **DisposeBag**，意味着 **DisposeBag** 对 **Disposable** 有一个强引用 。**Disposable** 强引用着 **Observable **以保证 **Observable** 的存活。而 **Observable ** 通过 **map**  操作符的闭包引用了 **UIViewController**。

最后**UIViewController**持有对  **DisposeBag** 的强引用，💥💥BOOM💥💥 … 循环引用了! 😱

如果你不能理解的话，建议你画一个图表

<img src="http://i1.wp.com/adamborek.com/wp-content/uploads/2017/06/retain_cycle_disposeBag.png?zoom=2&amp;w=1080"  />



## 如何避免循环引用？

我要说的是，一个良好的设计会使循环引用出现的几率越来越低。良好的关注点分离是关键。

上面的代码太少，不足以谈论架构模式。但如果摆脱循环引用呢？

只需要使用捕获列表 `capture list` !

使用捕获列表，可以传递变量，并告诉编译器闭包应如何处理该变量。通常，第一个想法是传递 `[weak self]`:

```swift
//[...]
let parsedObject = theObservable
    .map { [weak self] json in
        return self?.parser.parse(json) //compile-time error. What should be returned if `self` is nil?
    }
//[...]
```



然而，但使用 `[weak]` 时，我们需要告诉编译器，如果 `self`  为  `nil`  时应该返回什么。在这种情况下，最会是直接传递 `parser` ，而不是 `self`:

```swift
//[...]
let parsedObject = theObservable
    .map { [parser] json in
        return parser.parse(json)
    }
//[...]
```

swift 允许我们不使用像 `weak` 或 `unowned`这样的修饰符，直接向捕获列表传递变量。如果我们这样做，编译器就会知道只对 `parser` 进行强引用，而不是对 `self` 进行强引用。

就是这样， 一个小小的改变就可以解决整个问题！

> 第三个解决方法是使用的 `[unowned self]` 代替  `[weak self]`，然而，我并不是 `unowned` 的粉丝，它在某些情况下会导致崩溃😟。



## 使用 self 不等于循环引用

现在我们知道了每个操作符都会强引用闭包中所捕获的变量，包括 `self`。我要强调的是，不需要到处都避免使用 `self`。

如果你有一个类，只是返回一个 `Observable`，你完全可以放心地在创建Observable 的操作符闭包中捕获 `self`。

例如你的 `UIViewControlle`r 依赖一个 `APIClient`，你可以在 `APIClient` 的实现中使用 `self`。

 循环引用通常发生在 `self` 也是 `DisposeBag` 的所有者时。在任何其他情况下，在操作符闭包中使用 `self` 都是相当安全的。



## 总结

希望现在对你来说 `DisposeBag` 不再有什么神秘的地方。 它只是一个内部具有多个 `Disposable` 的数组。  在` deinit` 时` dispose` 所有 `Disposable`。 否则，我们的生活将更加艰难。

不幸的是，使用 `DisposeBag` 可能会导致内存泄漏。记住每个操作符的闭包都对其捕获的变量进行强引用。如果捕获到了持有 `Observable` 的 `self`，会导致循环引用。



如果将 `Disposable` 添加到 `DisposeBag` 中，则只需使用捕获列表并将适当的变量和属性传递给闭包即可。这样可以避免循环引用。

