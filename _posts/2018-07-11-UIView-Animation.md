---
layout: post
title: UIView Animation
subtitle: UIViewAnimation
date: 2018-07-11 16:09:16
categories: iOS
---
- animate
- transtion
- keyframe
<!-- more -->

## API 

```swift
@available(iOS 4.0, *)
open class func animate(withDuration duration: TimeInterval, delay: TimeInterval, options: UIView.AnimationOptions = [], animations: @escaping () -> Void, completion: ((Bool) -> Void)? = nil)

@available(iOS 4.0, *)
open class func animate(withDuration duration: TimeInterval, animations: @escaping () -> Void, completion: ((Bool) -> Void)? = nil)

@available(iOS 4.0, *)
open class func animate(withDuration duration: TimeInterval, animations: @escaping () -> Void)

@available(iOS 7.0, *)
open class func animate(withDuration duration: TimeInterval, delay: TimeInterval, usingSpringWithDamping dampingRatio: CGFloat, initialSpringVelocity velocity: CGFloat, options: UIView.AnimationOptions = [], animations: @escaping () -> Void, completion: ((Bool) -> Void)? = nil)

@available(iOS 4.0, *)
open class func transition(with view: UIView, duration: TimeInterval, options: UIView.AnimationOptions = [], animations: (() -> Void)?, completion: ((Bool) -> Void)? = nil)

@available(iOS 4.0, *)
open class func transition(from fromView: UIView, to toView: UIView,duration: TimeInterval, options: UIView.AnimationOptions = [],completion: ((Bool) -> Void)? = nil)

@available(iOS 7.0, *)
open class func perform(_ animation: UIView.SystemAnimation, on views: [UIView], options: UIView.AnimationOptions = [], animations parallelAnimations: (() -> Void)?, completion: ((Bool) -> Void)? = nil)

@available(iOS 7.0, *)
open class func animateKeyframes(withDuration duration: TimeInterval, delay: TimeInterval, options: UIView.KeyframeAnimationOptions = [], animations: @escaping () -> Void, completion: ((Bool) -> Void)? = nil)

@available(iOS 7.0, *)
open class func addKeyframe(withRelativeStartTime frameStartTime: Double, relativeDuration frameDuration: Double, animations: @escaping () -> Void)
```
## 参数介绍
- duration: 单次动画时长
- delay: 动画开始延时时间
- options: 动画配置选项，类型为 `UIView.AnimationOptions`, 默认为空即`[]`
- animations: 动画块，类型为`() -> Void`,再次闭包中更改 View 属性 来达到动画效果
- completion: 动画结束回调，类型为`((Bool) -> Void)?`,默认为 `nil`

### 可动画的属性
在 iOS 中使用 UIView Animation 可以非常容易地实现动画效果。UIView Animation 中可动画的属性有：

- **Position and size**
    - **bounds**
    - **frame**：
    - **center**

- **Appearance**
    - **backgroundColor**
    - **alpha**

- **Transformation**
    - **transform**

## options
- **repeat**
    - Repeat the animation indefinitely.
- **autoreverse**
    - Run the animation backwards and forwards (must be combined with the repeat option).
![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft604b1ovog20dc020764.gif)
- **curveEaseInOut**
    - Specify an ease-in ease-out curve, which causes the animation to begin slowly, accelerate through the middle of its duration, and then slow again before completing.
- **curveEaseIn**
    - An ease-in curve causes the animation to begin slowly, and then speed up as it progresses.
- **curveEaseOut**
    - An ease-out curve causes the animation to begin quickly, and then slow as it completes.
- **curveLinear**
    - A linear animation curve causes an animation to occur evenly over its duratio

![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft5psnb6vng20jg0gutfp.gif)

- **transitionFlipFromLeft**
    - A transition that flips a view around its vertical axis from left to right (the left side of the view moves toward the front and right side toward the back).

- **transitionFlipFromRight**
    - A transition that flips a view around its vertical axis from right to left (the right side of the view moves toward the front and left side toward the back).
- **transitionFlipFromTop**
    - A transition that flips a view around its horizontal axis from top to bottom (the top side of the view moves toward the front and the bottom side toward the back). 
- **transitionFlipFromBottom**
    - A transition that flips a view around its horizontal axis from bottom to top (the bottom side of the view moves toward the front and the top side toward the back).
- **transitionCurlUp**
    - A transition that curls a view up from the bottom. 
- **transitionCurlDown**
    - A transition that curls a view down from the top. 
- **transitionCrossDissolve**
    - A transition that dissolves from one view to the next. 

![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft5zzkw3b2g208w0fi7wi.gif)

## 简单动画
对于 UIView 上简单的动画，iOS 提供了很方便的函数。

```swift
class func animate(withDuration duration: TimeInterval, delay: TimeInterval, options: UIView.AnimationOptions = [], animations: @escaping () -> Void, completion: ((Bool) -> Void)? = nil)
```

## Spring 动画

iOS 还提供了对于 UIVIew 的 Spring 动画的 API

```swift
class func animate(withDuration duration: TimeInterval, delay: TimeInterval, usingSpringWithDamping dampingRatio: CGFloat, initialSpringVelocity velocity: CGFloat, options: UIView.AnimationOptions = [], animations: @escaping () -> Void, completion: ((Bool) -> Void)? = nil)
```
相对普通动画多了两个参数：
### **usingSpringWithDamping**

阻尼系数， 0 ~ 1之间，值越小动画弹性越明显，如果设置为 1，则动画不会有弹性效果。

在 initialSpringVelocity 为 0 ，damping 分别为 0.4，0.6，0.8 的情况下效果如下图：
![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft5yprawk3g20aw0c8t9h.gif)

### **initialSpringVelocity**

视图在动画开始时的速度，相同阻尼系数下，速度越大弹动的幅度越大。

在 damping 为 1 ，initialSpringVelocity 分别为 0，5，30 的情况下效果如下图：

![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft5yt2qw12g20a80c00tm.gif)

## transition 动画
iOS 还提供了过渡动画的 API, 需要注意的是，换场动画会在这两个 View 共同的父 View 上进行，在写动画之前，先要设计好 View 的继承结构。

```
open class func transition(with view: UIView, duration: TimeInterval, options: UIView.AnimationOptions = [], animations: (() -> Void)?, completion: ((Bool) -> Void)? = nil)

@available(iOS 4.0, *)
open class func transition(from fromView: UIView, to toView: UIView, duration: TimeInterval, options: UIView.AnimationOptions = [], completion: ((Bool) -> Void)? = nil)
```

View 之间的转换也有很多选项可选，例如 UIViewAnimationOptionTransitionFlipFromLeft 从左边翻转，UIViewAnimationOptionTransitionCrossDissolve 渐变等等。

### keyframe 动画
keyframe 动画即关键帧动画，上面的那谢动画我们只能控制开始和结束时的效果，然后由系统补全中间的过程，有些时候我们需要自己设定若干关键帧，实现更复杂的动画效果。
```
@available(iOS 7.0, *)
open class func animateKeyframes(withDuration duration: TimeInterval, delay: TimeInterval, options: UIView.KeyframeAnimationOptions = [], animations: @escaping () -> Void, completion: ((Bool) -> Void)? = nil)

@available(iOS 7.0, *)
open class func addKeyframe(withRelativeStartTime frameStartTime: Double, relativeDuration frameDuration: Double, animations: @escaping () -> Void)
```

下面是一个范例：

```
UIView.animateKeyframes(withDuration: 2, delay: 0.0, options: [.repeat, .autoreverse,], animations: {
            UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.5, animations: {
                self.redView.frame = CGRect(x: 10, y: 50, width: 100, height: 100)
            })
            
            UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.3, animations: {
                self.redView.frame = CGRect(x: 20, y: 100, width: 100, height: 100)
            })
            
            UIView.addKeyframe(withRelativeStartTime: 0.8, relativeDuration: 0.2, animations: {
                self.redView.transform = CGAffineTransform(scaleX: 0.5, y: 0.5)
            })
        }, completion: nil)
```

动画时长为 `2` 秒，在 `0` 到 `(0.5 * 2)` 秒时，将 redView 的 frame 动画到 `CGRect(x: 10, y: 50, width: 100, height: 100)`, 在 `(0.5 * 2)` 到 `(0.5 + 0.3) * 2`秒 时，将 redView 的 frame 动画到 `CGRect(x: 20, y: 100, width: 100, height: 100)`，在 `0.8` 到 `(0.8 + 0.2) * 2` 秒时，对其进行缩放，即宽高变为原为原来的 0.5 倍。

这里用到了 `CGAffineTransform`, 它是用来对 View 到 2D 变换的。之后再介绍吧。
