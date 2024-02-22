---
layout: post
title: UIBeizerPath简介
subtitle: UIBeizerPath简介
date: 2018-04-25 16:15:25
categories: iOS
---

<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9gy1fqoz05odbdg20d50aiu0x.gif"/>
</p>
</html>

<!-- more -->

```
A path that consists of straight and curved line segments that you can render in your custom views.
```

`UIBezierPath`(贝塞尔曲线),位于 UIKit 框架,使用此类可以定义简单的形状,如椭圆或者矩形,或者有多个直线和曲线段组成的形状.我们可以使用直线段去创建矩形和多边形,使用曲线段去创建弧(arc),圆或者其他复杂的曲线形状. 曲线定义: 起始点,终止点(锚点),控制点.通过调整控制点,贝塞尔曲线会发生变化.

## 初始化 UIBezierPath

系统提供了多种初始化 `UIBezierPath` 的方法:

```
    // 返回一个矩形 path
    public convenience init(rect: CGRect)
    
    // 返回一个圆形或者椭圆形 path
    public convenience init(ovalIn rect: CGRect)

    // 返回一个带圆角的矩形 path ,矩形的四个角都是圆角;
    public convenience init(roundedRect rect: CGRect, cornerRadius: CGFloat) 
    
    // 返回一个带圆角的矩形 path , UIRectCorner 指定绘制哪些圆角;
    public convenience init(roundedRect rect: CGRect, byRoundingCorners corners: UIRectCorner, cornerRadii: CGSize)

    // 返回一段圆弧,参数说明: center: 弧线中心点的坐标 radius: 弧线所在圆的半径 startAngle: 弧线开始的角度值 endAngle: 弧线结束的角度值 clockwise: 是否顺时针画弧线.
    public convenience init(arcCenter center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat, clockwise: Bool)

    // 用一条 CGpath 初始化
    public convenience init(cgPath CGPath: CGPath)
    
    // 初始化方法,需要用实例方法添加线条.使用比较多,可以根据需要任意定制样式,画任何我们想画的图形.
    public init()
    
    // 返回一个反转当前路径的路径对象.(反方向绘制path)
    open func reversing() -> UIBezierPath // Modified paths
```

## 创建 UIBezierPath

### 1. 矩形 Path
```
    let path = UIBezierPath(rect: CGRect(x: 100, y: 400, width: 100, height: 50))
    path.lineWidth = 2 // 线条宽度
    path.stroke()
```

<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/mw690/b92f96b9gy1fqoq8f4o2fj20790egaa5.jpg"/>
</p>
</html>

### 2. 椭圆 path

```
    // 宽高相等为圆， 不等则为椭圆
    let path = UIBezierPath(ovalIn: CGRect(x: 100, y: 400, width: 100, height: 100))
    path.lineWidth = 2
    path.stroke()

```

<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/mw690/b92f96b9gy1fqoqg6hilyj20750e7mxb.jpg"/>
</p>
</html>

### 3. 圆角矩形 path

```
    let path = UIBezierPath(roundedRect: CGRect(x: 100, y: 400, width: 100, height: 80), cornerRadius: 20)
    path.lineWidth = 2
    path.stroke()
```
<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/mw690/b92f96b9gy1fqoqmrmh0ij20740e8jri.jpg"/>
</p>
</html>

### 4. 指定圆角的矩形 path

```
    let path = UIBezierPath(roundedRect: CGRect(x: 100, y: 400, width: 100, height: 80), byRoundingCorners: [.bottomLeft, .topRight], cornerRadii: CGSize(width: 20, height: 20))
    path.lineWidth = 2
    path.stroke()
```

<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/mw690/b92f96b9ly1fqoqq9zkubj20750eiwel.jpg"/>
</p>
</html>

### 5. 一段圆弧 path

```
    let path = UIBezierPath(arcCenter: CGPoint(x: 150, y: 400), radius: 50, startAngle: 0, endAngle: CGFloat.pi * 0.5, clockwise: false)
    path.lineWidth = 2
    path.stroke()
```

<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/mw690/b92f96b9ly1fqotnmqgtdj206v0e1t8u.jpg"/>
</p>
</html>

其中角度示意图如下：

![](https://ws1.sinaimg.cn/large/b92f96b9ly1fqotrk7jzdj20b40b40tr.jpg)

### 6. 自定义 path

```
    let path = UIBezierPath()
    path.lineWidth = 2
    
    let center = CGPoint(x: 150, y: 400)
    path.move(to: center)
    path.addArc(withCenter: center, radius: 50, startAngle: CGFloat.pi / 6.0, endAngle: CGFloat.pi * 11 / 6.0, clockwise: true)
    path.close()
    path.stroke()
```
<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fqoulzwmhgj206y0e4wen.jpg"/>
</p>
</html>


## 构建 Path

上面前 5 种，使用了系统方法创建 path，第 6 种则自定义了简单的 path，创建自定义 path, 会用到以下方法：

```
    // 以 point点 开始作为起点, 一般用`UIBezierPath()`创建的贝塞尔曲线，先用该方法标注一个起点，再调用其他的创建线条的方法来绘制曲线
    open func move(to point: CGPoint)
    
    // 绘制二次贝塞尔曲线的关键方法,即从path的最后一点开始添加一条线到point点
    open func addLine(to point: CGPoint)
    
    e.g. ![](https://ws1.sinaimg.cn/large/b92f96b9ly1fqovkdwhkoj206203ha9t.jpg)
        let path = UIBezierPath()
        path.lineWidth = 2
        path.move(to: CGPoint(x: 100, y: 400))
        path.addLine(to: CGPoint(x: 200, y: 400))
        path.stroke()
    
    // 绘制二次贝塞尔曲线的关键方法,和`-moveToPoint:`配合使用. endPoint为终止点,controlPoint为控制点.
    open func addQuadCurve(to endPoint: CGPoint, controlPoint: CGPoint)
    
    e.g. ![](https://ws1.sinaimg.cn/large/b92f96b9gy1fqovj22qqvj206i06pglg.jpg)
        let path = UIBezierPath()
        path.lineWidth = 2
        path.move(to: CGPoint(x: 100, y: 400))
        path.addLine(to: CGPoint(x: 200, y: 400))
        path.addQuadCurve(to: CGPoint(x: 150, y: 600), controlPoint: CGPoint(x: 150, y: 450))
        path.stroke()
    
    // 绘制三次贝塞尔曲线的关键方法,以三个点画一段曲线. 一般和moveToPoint:配合使用.
    // 其中,起始点由`-moveToPoint:`设置,终止点位为`endPoint:`, 控制点1的坐标controlPoint1,控制点2的坐标是controlPoint2.
    open func addCurve(to endPoint: CGPoint, controlPoint1: CGPoint, controlPoint2: CGPoint)
    
    e.g. ![](https://ws1.sinaimg.cn/large/b92f96b9gy1fqovs2biatj205i08d3ye.jpg)
        let path = UIBezierPath()
        path.lineWidth = 2
        
        path.move(to: CGPoint(x: 230, y: 140))
        path.addCurve(to: CGPoint(x: 230, y: 415), controlPoint1: CGPoint(x: 145, y: 275), controlPoint2: CGPoint(x: 340, y: 275))
        path.stroke()
    
    // 闭合路径
    open func close()
    
    // 移除所有的点,从而有效地删除所有子路径
    open func removeAllPoints()
    
    // 追加 Path 到路径
    open func append(_ bezierPath: UIBezierPath)
    
    // 用仿射变换矩阵变换路径的所有点
    open func apply(_ transform: CGAffineTransform)
```

## path 信息

```
    // A Boolean value indicating whether the path has any valid elements.
    // 该值指示路径是否有任何有效的元素
    open var isEmpty: Bool { get }
    
    // The bounding rectangle of the path.
    // 路径包括的矩形
    open var bounds: CGRect { get }
    
    // The current point in the graphics path.
    // 图形路径中的当前点
    open var currentPoint: CGPoint { get }
    
    // Returns a Boolean value indicating whether the area enclosed by the receiver contains the specified point.
    // 接收器是否包含指定的点
    open func contains(_ point: CGPoint) -> Bool
```
## 绘制属性

```
    // The line width of the path.
    // 线宽
    open var lineWidth: CGFloat

    // The shape of the paths end points when stroked.
    // 绘制时端点类型
    // .butt
    // .round 圆角
    // .suqare 方形，样式上和 .butt是一样的，但是比 .butt 长一点
    open var lineCapStyle: CGLineCap

    // The shape of the joints between connected segments of a stroked path.
    // 线段连接类型
    // .miter 尖角
    // .round 圆角
    // .bevel 斜角
    open var lineJoinStyle: CGLineJoin
    
    // The limiting value that helps avoid spikes at junctions between connected line segments
    // 最大斜接长度（只有在使用kCGLineJoinMiter是才有效,最大限制为10）， 边角的角度越小，斜接长度就会越大，为了避免斜接长度过长，使用lineLimit属性限制，如果斜接长度超过miterLimit，边角就会以KCALineJoinBevel类型来显示
    open var miterLimit: CGFloat // Used when lineJoinStyle is kCGLineJoinMiter

    // The factor that determines the rendering accuracy for curved path segments.
    // 决定曲线路径段渲染精度
    open var flatness: CGFloat
    
    // A Boolean indicating whether the even-odd winding rule is in use for drawing paths.
    //  判断奇偶数组的规则绘制图像,图形复杂时填充颜色的一种规则。类似棋盘
    // Default is NO. When YES, the even-odd fill rule is used for drawing, clipping, and hit testing.
    open var usesEvenOddFillRule: Bool 

    // Sets the line-stroking pattern for the path.
    // 设置路径的线条描边模式
    // 为path绘制虚线，dash数组存放各段虚线的长度，count是数组元素数量，phase是起始位置
    open func setLineDash(_ pattern: UnsafePointer<CGFloat>?, count: Int, phase: CGFloat)

    open func getLineDash(_ pattern: UnsafeMutablePointer<CGFloat>?, count: UnsafeMutablePointer<Int>?, phase: UnsafeMutablePointer<CGFloat>?)
```
## 图形上下文中的路径操作

```
    // Paints the region enclosed by the receiver’s path using the current drawing properties.
    // 填充路径闭合的区域
    func fill()
    
    // Paints the region enclosed by the receiver’s path using the specified blend mode and transparency values.
    // 填充模式, alpha 设置
    // blendMode : https://onevcat.com/2013/04/using-blending-in-ios/
    func fill(with: CGBlendMode, alpha: CGFloat)
    
    // Draws a line along the receiver’s path using the current drawing properties.
    各个点连线
    func stroke()
    
    // Draws a line along the receiver’s path using the specified blend mode and transparency values.
    // 链接模式, alpha 设置
    func stroke(with: CGBlendMode, alpha: CGFloat)
    
    // Intersects the area enclosed by the receiver’s path with the clipping path of the current graphics context and makes the resulting shape the current clipping path.
    // 图形绘制超出当前路径范围,则不可见
    func addClip()

```

## 显示到屏幕上

`UIBezierPath` 生成的是矢量图形，不能直接显示在屏幕上，如果想让其展示在屏幕上，有两种方法：

1. 在 UIView 的 `func draw(_ rect: CGRect) `中绘制，并设置颜色

```
    override func draw(_ rect: CGRect) {
        UIColor.orange.set()
        
        let path = UIBezierPath()
        path.lineWidth = 2
        
        path.move(to: CGPoint(x: 230, y: 140))
        path.addCurve(to: CGPoint(x: 230, y: 415), controlPoint1: CGPoint(x: 145, y: 275), controlPoint2: CGPoint(x: 340, y: 275))
        path.stroke()
    }
```

2.将 一个 `layer` 的 `path` 指向 `UIBezierPath` 实例的 `cgPath`:

```
    let path = UIBezierPath(roundedRect: CGRect(x: 100, y: 100, width: 100, height: 100), cornerRadius: 50)
        
    let shapeLayer = CAShapeLayer()
    shapeLayer.path = path.cgPath
    shapeLayer.fillColor = UIColor.orange.cgColor
    shapeLayer.strokeColor = UIColor.green.cgColor
    view.layer.addSublayer(shapeLayer)
```

## 应用

给 UIView 添加圆角，以 UIImageView 为例

```
    let path = UIBezierPath(roundedRect: imageView.bounds, cornerRadius: 0)
    let round = UIBezierPath(arcCenter: CGPoint(x: imageView.bounds.width / 2, y: imageView.bounds.height / 2), radius: imageView.bounds.width / 2, startAngle: 0, endAngle: CGFloat.pi * 2, clockwise: false)
    path.append(round) // path 包含的区域如下图红色部分
    
    
    let shapeLayer = CAShapeLayer()
    shapeLayer.path = path.cgPath
    shapeLayer.fillColor = UIColor.red.cgColor
    imageView.layer.borderColor = UIColor.clear.cgColor
    imageView.layer.addSublayer(shapeLayer)
```
<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9ly1fqoyaqdkhzj206l06zt9t.jpg"/>
</p>
</html>

要达到圆角效果需要将 `fillColor` 和 imageView 的 `superView` 的颜色保持一致, 即：
```
    shapeLayer.fillColor = UIColor.red.cgColor
```
最终效果如下：

<html> 
<p align="center">
<img src="https://ws1.sinaimg.cn/large/b92f96b9gy1fqoyfrn9ufj207u08edgz.jpg"/>
</p>
</html>

## Reference

- [UIBezierPath](https://developer.apple.com/documentation/uikit/uibezierpath)
- [贝塞尔曲线扫盲-前端乱炖-http://www.html-js.com/article/1628](http://www.html-js.com/article/1628)
- [iOS UIBezierPath(贝塞尔曲线)](https://www.jianshu.com/p/b561e208f51f)
