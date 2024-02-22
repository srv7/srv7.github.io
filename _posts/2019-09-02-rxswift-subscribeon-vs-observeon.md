---
layout: post
title: RxSwift - subscribeOn vs observeOn
date: 2019-09-02 10:15:11
subtitle: rxswift-subscribeon-vs-observeon
categories: Rx
---

RxSwift 中线程切换是比较方便的，但其中也有值得注意的一些事项。

##  __Observable subscriptions__
订阅（`subscribing`）和观察（`observing`）方面的术语有点混乱，所以让我们首先解决这个问题。

<!-- more -->

让我们看一下可观察订阅的工作原理。我们可以将订阅分成3部分：

![observable subscription](http://rx-marin.com/images/subscribe-schema.png)

1. 首先定义了一个 `Observable`, 在某些情况下，在闭包中提供一些代码来执行工作并向任何观察者发出元素。当你在创建 `Observable` 的时候，这些代码并不是立即执行，而是存储起来以备后用。如果没有 `observers`, `observable` 将不会做任何事，直到有 `observer` 来订阅他。
2. 在为订阅建模时，您可以使用一些运算符（如map，filter等）来处理发出的元素。添加运算符仍然不执行任何工作, 只是创建一个“更专业”的 `Observable。`
3. 只有当在 一个 `Observable` 上调用 `subscribe(...)` 方法时，才“打开了”一个 `Observable`。`subscribe(...)`方法调用后，1 中 定义的代码才真正的执行。

这里有两个要点：
- __subscription code__ 是调用 `subscribe()`方法调用后被执行的代码， 位于 __Observable.create { ... }__ 的闭包内。它是创建 订阅（subscription）和产生元素的代码。
- __observation code__ 是观察发射出来的元素的代码，是在 __onNext: { ... }, onCompleted: {...}__ 中提供的代码。这里是你观察元素的地方。

![subscription code vs. observation code](http://rx-marin.com/images/subscribe-terms.png)

## __线程切换__
### subscribeOn：
此操作符将更改 __subscription code__ 的执行线程。subscribeOn 将会改变无论是在它之前调用还是之后调用的方法的执行线程，无关顺序。 subscribeOn 可以放在任何位置（有例外，见下文）。

### observeOn:
此操作符将会更改 __observer code__ 的执行线程。

### __observeOn 只工作在下游__

```swift
let diseposeBag = DisposeBag()

Observable<Int>.create { observer in
  observer.onNext(1)
  sleep(1)
  observer.onNext(2)
  return Disposables.create()
}
.map { value -> Int in
  print("\n\n 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 2
}
.observeOn(SerialDispatchQueueScheduler(qos: .background))
.map { value -> Int in
  print(" 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 3
}
.observeOn(MainScheduler.instance)
.subscribe(onNext: { element in
  print(" 😀 Queue: \(element) \(self.currentQueueName() ?? "queue")")
}).disposed(by: disposeBag)
```
![](https://miro.medium.com/max/1312/1*nbkfImml070K1JFD1Yb5gQ.png)

### __subscribeOn位置无关紧要。它工作在下游和上游__
```swift
Observable<Int>.create { observer in
  observer.onNext(1)
  sleep(1)
  observer.onNext(2)
  return Disposables.create()
}
.map { value -> Int in
  print("\n\n 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 2
}
.subscribeOn(SerialDispatchQueueScheduler(qos: .background))
.map { value -> Int in
  print(" 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 3
}
.observeOn(MainScheduler.instance)
.subscribe(onNext: { element in
  print(" 😀 Queue: \(element) \(self.currentQueueName() ?? "queue")")
}).disposed(by: disposeBag)
```
![](https://miro.medium.com/max/1346/1*IjPOSzd9Gs3XW5VcaqB2Sw.png)

### __连续的 subscribeOn 不会连续改变线程__
```swift
Observable<Int>.create { observer in
  observer.onNext(1)
  sleep(1)
  observer.onNext(2)
  return Disposables.create()
}
.subscribeOn(SerialDispatchQueueScheduler(qos: .background))
.map { value -> Int in
  print("\n\n 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 2
}
.subscribeOn(MainScheduler.instance)
.map { value -> Int in
  print(" 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 3
}
.observeOn(MainScheduler.instance)
.subscribe(onNext: { element in
  print(" 😀 Queue: \(element) \(self.currentQueueName() ?? "queue")")
}).disposed(by: disposeBag)
view raw
```

### __连续的 observeOn 将连续改变线程__
```swift
Observable<Int>.create { observer in
  observer.onNext(1)
  sleep(1)
  observer.onNext(2)
  return Disposables.create()
}
.observeOn(ConcurrentDispatchQueueScheduler(qos: .background))
.map { value -> Int in
  print("\n\n 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 2
}
.observeOn(SerialDispatchQueueScheduler(qos: .background))
.map { value -> Int in
  print(" 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 3
}
.observeOn(MainScheduler.instance)
.subscribe(onNext: { element in
  print(" 😀 Queue: \(element) \(self.currentQueueName() ?? "queue")")
}).disposed(by: disposeBag)
```

### __subscribeOn 无法覆盖 observeOn 的线程更改__

```swift
Observable<Int>.create { observer in
  observer.onNext(1)
  sleep(1)
  observer.onNext(2)
  return Disposables.create()
}
.observeOn(SerialDispatchQueueScheduler(qos: .background))
.map { value -> Int in
  print("\n\n 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 2
}
.subscribeOn(MainScheduler.instance)
.map { value -> Int in
  print(" 😀 Queue: \(self.currentQueueName() ?? "queue")")
  return value * 3
}
.observeOn(MainScheduler.instance)
.subscribe(onNext: { element in
  print(" 😀 Queue: \(element) \(self.currentQueueName() ?? "queue")")
}).disposed(by: disposeBag)
```
![](https://miro.medium.com/max/1262/1*ptKZtkVuvpi-67aqb7cv9w.png)


## currentQueueName
```swift
func currentQueueName() -> String? {
    let name = __dispatch_queue_get_label(nil)
    return String(cString: name, encoding: .utf8)
}
```

## 引用
- [observeOn vs. subscribeOn - rx_marin](http://rx-marin.com/post/observeon-vs-subscribeon/)
- [RxSwift: subscribeOn vs observeOn? - Aaina jain - swift-india](https://medium.com/swift-india/rxswift-subscribeon-vs-observeon-2b121ba95161)