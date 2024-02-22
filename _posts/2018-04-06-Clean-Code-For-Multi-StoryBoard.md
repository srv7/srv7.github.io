---
layout: post
title: Clean Code for Multiple Storyboards
date: 2018-04-06 15:34:47
categories: iOS
published: false
---

虽然 storyboard 在 iOS5中被推出，但许多开发者都不愿意使用它，尤其是在大型项目中。 尽管我比较喜欢 storyboard，但我也必须承认许多开发者鄙视 storyboard 并不是完全是错误的。

<!-- more -->

**问题1**： 对于大型项目，故事板经常变得笨重和难以管理。  
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7chjui5vj20dw0dln2l.jpg)

**问题2**：故事板非常慢。随着规模的扩大，它们不仅对开发人员而且对XCode也变得不那么方便。一旦你点击一个这样大小的故事板文件，你就可以走出你的座位，在餐厅漫步，喝一杯咖啡，向大家打招呼，然后回来。如果XCode没有被卡住并且已成功加载故事板，那么您将会很幸运。

**问题3**：在版本控制中合并冲突。当多个开发人员在进行项目时，总是会有一种痛苦的体验，因为它几乎总是会导致故事板中的合并冲突。

**问题4**：人们永远无法专注于应用程序的特定功能。

**问题5**：没听别的开发者说起过，但在我自己的项目中遇到了这个问题。使用巨大的故事板导致应用程序的加载非常缓慢。将它拆分为多个故事板可以解决我的问题。

---

尽管存在上述问题，我觉得我们可以以一种很好，干净的方式使用故事板，来解决大部分上述问题。我也觉得有一个像上面那样大的故事板，并没有像它失败那样达到它的目的。故事板是为了解释故事，而不是传奇。一个应用程序的故事板可以很容易地分成多个故事板，每个故事板都代表一个单独的故事。最好的例子是预登录流程。启动画面，登录，注册，忘记密码，T＆C(terms and conditions)等可以很容易地被视为一个独立的流程，独立于应用程序的功能。  

多个故事板解决了问题1、2、4和5，因为解决方案是专门为此设计的。问题3在很大程度上得到了解决，因为只有当多个开发人员在同一故事板上工作时才会发生合并冲突。你可以找到许多[博客](http://www.newventuresoftware.com/blog/organizing-xcode-projects-using-multiple-storyboards)，教程使用多个故事板赞同这个想法。  

苹果也提供了它对这一想法的认可，并在iOS 9 中引入了故事板参考。这里有一个很好的[教程](https://www.raywenderlich.com/115697/ios-9-storyboards-tutorial-whats-new-in-storyboards)，所以我不再重复了。

> Enters Linus Torvalds : “Talk is cheap, Show me the code!”  
ME : OK Boss 😏


**TL;DR (Too Long; Didn't Read)**

**在一个项目中使用多个故事板是一个好主意，让我们看看如何以一种干净的方式完成它。**

**快速浏览我们将要实现的目标。**

![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7d57frbaj20m80210tg.jpg)
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7d61spnjj20m802mwey.jpg)

## step 1： AppStoryboard 枚举
```
enum AppStoryboard : String {
    case Main = "Main"
    case PreLogin = "PreLogin"
    case Timeline = "Timeline"
    var instance : UIStoryboard {
      return UIStoryboard(name: self.rawValue, bundle: Bundle.main)
    }
}
// USAGE :

let storyboard = AppStoryboard.Main.instance

// Old Way

let storyboard = UIStoryboard(name: “Main”, bundle: Bundle.main)
```

AppStoryboard 是一个具有原始类型为 String 的枚举，因为 Storyboard 是通过其名称(即字符串字面量)实例化的。此枚举应根据情况列出项目中所有的 storyboard 作为他的 cases 。它更像是项目中故事板的命名空间。使用这样的枚举，开发人员/代码审阅者可以方便地查看项目中的所有故事板。更重要的是，它可以避免使用字符串字面量来实例化故事板，并且可以利用自动完成功能来获得故事板实例。没有打字的余地！  

我总是尽最大努力避免代码中的字符串文字，因为编译器不会警告字符串文字中的键入错误，这会导致恼人的错误和崩溃。你永远不要低估人类犯错的能力！  

如果您约定使用与enum中case名称相同的 storyboard 要名称，则可以以更简洁的方式重新编写enum。可以忽略原始值，因为对于字符串类型枚举，大小写名称是隐式原始值。

```
enum AppStoryboard : String {
    case Main, PreLogin, Timeline
    var instance : UIStoryboard {
      return UIStoryboard(name: self.rawValue, bundle: Bundle.main)
    }
}
```

通过使用 AppStoryboard 枚举， 你可以像下面一样实例化 storyboard：
```
let mainStoryboard = AppStoryboard.Main.instance
let preLoginStoryboard = AppStoryboard.PreLogin.instance
let timelineStoryboard = AppStoryboard.Timeline.instance
// Instantiate ViewController
let loginScene = preLoginStoryboard.instantiateViewController(
                                         withIdentifier: "LoginVC")
// OR One-liner
let loginScene = AppStoryboard.PreLogin.instance
               .instantiateViewController(withIdentifier: "LoginVC")
```

至少比我们早些时候需要做的要好。对。

## step 2：扩展 UIViewController

每次实例化 ViewController 时都要写 storyboard identifier 也容易出错，因为它包括字符串字面量。现在，也是摆脱这种状况的时候了。让我们在 ViewControllers 中放置一个静态计算属性，如下所示:
```
static var storyboardID : String {
     return "<Storyboard Identifier>"
}
```
现在你可以这样拿到 storyboard identifier：

```
<ViewController_Name>.storyboardID
// e.g. LoginVC.storyboardID
```

但不太方便，大多数开发人员都使用标准做法，对ViewController类和storyboard identifier 使用相同的名称，或者在名称后面附加后缀，即“. ID”。这是一个很好的实践，因为通过使用此方法扩展UIViewController类，它可以避免在项目中的所有视图控制器中放置静态计算属性。让我们看看怎么做。

```
extension UIViewController {
   class var storyboardID : String {
     return "\(self)"
   }
}
```
> 请注意，我使用“class”而不是“static”，因为static属性/方法[不能被子类覆盖](http://stackoverflow.com/questions/29636633/static-vs-class-functions-variables-in-swift-classes)，以提供自定义故事板标识符（如果需要）。
> 
> 有人可能会试图使用Swift Reflection API，以便在扩展中获得正确的类名称，这在这里简单地通过使用“self”来实现。[替代方案](https://gist.github.com/Gurdeep0602/4fc3892c1b2861d4cd2062ddfddf3262)

现在，可以这样实例化 ViewController：

```
let loginScene = AppStoryboard.PreLogin
                 .viewController(viewControllerClass: LoginVC.self)
```

**优点**：我字符串字面量，XCode 自动补全  
**缺点**：太啰嗦，类型推断为 UIViewController 而不是 LoginVC

我不喜欢的是，这里 loginScene 被推断为 UIViewController 类型。因此，为了访问LoginVC的属性，必须强制转换它。

```
let loginScene = AppStoryboard.PreLogin
                 .viewController(viewControllerClass: LoginVC.self)                   
                 as! LoginVC
```

其次，它很长了，而且不得不进行类型强制转换。所以现在，让我们来修复类型转换，让它变得优雅一些。
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7e50adiuj20m80chwgj.jpg)

## Taking it to next level

让我们通过在 AppStoryboard中放置另一个方法，让AppStoryboard自己实例化视ViewControllers。
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7eamqwfbj20m802iaae.jpg)

此函数将类名作为参数并返回其实例。使用此方法，可以将视图控制器实例化为:
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7ebp6vbvj20m800z3yk.jpg)

这种写法虽然简短了，但依然没有解决类型强制转换问题。为此，需要用到[泛型](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html), 使用泛型参数和返回值重写该方法。

所以现在 AppStoryboard  看起来应该是这样
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7efs5mqbj20m80bjjsn.jpg)

T 是泛型参数，T.Type 是指 T 的数据类型。现在可以这样初始化 ViewController:
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7eibwl57j20m800zglo.jpg)

为了完整起见，我们还要在AppStoryboard枚举中包装UIStoryboard的“instantiateInitialViewController()”方法。
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7h1z3rs5j20m80f040g.jpg)

太好了。这就是我们对AppStoryboard枚举所能做的一切，但由于我仍然对结果不满意，我决定再进一步。因此，回到UIViewController的扩展。

在UIViewController的扩展中创建实例化视图控制器的静态方法将真正简化视图控制器的实例化过程。让我们试一试。
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7h4lzyxlj20m808f0th.jpg)

> 注意返回类型，即“Self”。self是指协议内部当前“对象”的类型。我很高兴看到它在一个扩展中也起作用。

现在可以实例化视图控制器，如下所示:
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7h6lc077j20m801c0sv.jpg)

太好了！！！这看起来漂亮多了。是我想要的方式。

由于该方法接受一个类型为 AppStoryboard 的枚举作为参数，可以利用 Swift 的类型推断将参数由`AppStoryboard.PreLogin` 简化为`.PreLogin`

完整代码见 [gist](https://gist.github.com/Gurdeep0602/4fc3892c1b2861d4cd2062ddfddf3262)


原文：[Clean Code for Multiple Storyboards](https://medium.com/@gurdeep060289/clean-code-for-multiple-storyboards-c64eb679dbf6)
