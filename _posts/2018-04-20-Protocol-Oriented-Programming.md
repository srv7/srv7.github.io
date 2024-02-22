---
layout: post
title: 使用协议实现 Loading 视图
subtitle: Loading
date: 2018-04-20 13:08:34
categories: iOS
---

## 需求
App 中的大多数页面在某些时候需要异步加载数据，异步加载数据需要花费一些时间并且可能会失败。为了让用户了解当前的活动状态，通常我们希望在加载数据的时候展示某种形式的活动指示器(`activity indicator`)，并在操作失败的情况下显示错误视图。

<!-- more -->

## Common Solution

### 继承

最基本的解决方案是创建一个`BaseViewController`, 其他的 `ViewController`都集成自`BaseViewController`。just like:  

```swift
class BaseViewController: UIViewController {
    func showActivityIndicator(visible: Bool) {
        ...
    }
    
    func handle(_ error: Error) {
        ...
    }
}
```
看起来没什么问题，使用起来很方便。但这样一来 `BaseViewController` 很容易成为各种通用功能的通用工具，以后维护起来可就有些麻烦了。  

使用 `BaseViewController` 来实现需求的另一个问题是，项目中的 Controller 只能继承自 `BaseViewController`, 如果我们要实现基于一个UITableView的视图控制器，继承 `UITableViewController` 可能会是一个更好的选择。`UITableViewController` 继承自 `UIVIewController` 而不是 `BaseViewController`, 这时该怎么做呢？再定义一个 `BaseTableViewController`? 这样的话就不够优雅了，需要将 `BaseViewController` 中的代码 copy 到 `BaseTableViewController`. 肯定不能这么做。

### 使用 extension

既然 `UIKit` 中的所有的试图控制器都是 `UIVIewController` 的子类，那就不妨将通用方法放到 `extension` 中:

```
extension UIViewController {
    func showActivityIndicator(visible: Bool) {
        ...
    }
    
    func handle(_ error: Error) {
        ...
    }
}
```
使用了 `extension`将通用功能从 `BaseViewController` 中剥离出来，不同的功能模块使用 `extension` 达到代码分离。

这样一来 App 中所有的视图控制器都添加了相应的功能。但是我们并不想像 `UINavigationController` 和 `UITabbarController` 这样的容器控制器添加这些功，那怎么做呢？

## 使用协议

当苹果在2015年的 WWDC 上发布 Swift 2 时，还声明了 Swift 是世界上第一个面向协议的编程语言。从他的名字我们可能认为`Protocol-Oriented Programming`全是关于协议的，但那是不对的。他不仅是编写应用程序的一种新方式，还是我们对编程的一种思考方式，而不仅仅是协议。


### 创建协议

```swift
// 1
public protocol LoadingRetryType where Self: UIViewController {
    // 2
    associatedtype Loading where Loading: UIViewController
    associatedtype Error where Error: UIViewController
    // 3
    var loadingView: Loading { get set }
    var errorView: Error { get set } 
}
// 4
public extension LoadingRetryType {
    func setLoadingView(visible: Bool) {
        visible ? self.add(loadingView) : loadingView.remove()
    }
    
    func setErrorView(visible: Bool) {
        visible ? self.add(errorView) : errorView.remove()
    }
}

```
note:
> 1. 创建了一个名为 `LoadingRetryType` 的协议，并限定了遵循改协议的类型是 `UIVIewController` 的子类.    
> 2. 声明了两个关联类型并限定了其类型是 `UIVIewController` 及其子类
> 3. 声明了两个遵循协议的类型必须实现的两个 `requirement`，类型分别是上面关联类型
> 4. 扩展 `LoadingRetryType` 协议，添加两个方法并添加默认实现

 一般来说两个 requirement 的类型应该是 `UIView` 才对, 这里为什么是  `UIVIewController` 呢？因为我想用`childViewController` 来控制 loading 和 retry 两个 View 的显示内容。实现协议的 Controller 来决定使用哪些 ViewController 来作为 loading 和 retry。
 
 > talking is cheap, show you the [code](https://github.com/srv7/LoadingRetry)
 

## Reference
- [using child view controllers as plugins in swift](https://www.swiftbysundell.com/posts/using-child-view-controllers-as-plugins-in-swift)
- [Swift 4 Protocol-Oriented Programming - Third Edition](https://www.packtpub.com/web-development/swift-protocol-oriented-programming-third-edition)
