---
layout: post
title: swift 4.1 遇到的问题
subtitle: swift 4.1
date: 2018-04-10 13:58:37
categories: swift
---

苹果前几天发布了 swift 4.1，相对于 swift 4.0 并没有太大变动。但有的三方库确出了问题，比如用于 JSON 解析的 HandyJSON。作者的解释是：

<!-- more -->

>swift4.1调整了类描述结构的内存布局，有个字段的含义改了，偏移量需要重新计算。  

看了下 fix 后的源码，之后几行改动

```swift
#if swift(>=4.1) || (swift(>=3.3) && !swift(>=4.0))
return NominalTypeDescriptor(pointer: relativePointer(base: base, offset: base.pointee - base.hashValue))
#else
return NominalTypeDescriptor(pointer: relativePointer(base: base, offset: base.pointee))
#endif
```

我的项目中并没有用到 HandyJSON，而是 用的了 swift4.0 中推出的 Codable 来解析 JSON。由于 swift4.1 相对于 swift4.0 的改动不大，理论上应该可以无痛过渡到 4.1，但不幸的是依然出了问题。出问题的地方恰恰是 JSON 解析这一块😂。说好的无痛呢😭😭

由于某些原因,项目中的部分 model 用的是 `class`，其他的则是用的 `struct`. 看了下 xcode 的报错信息，错误都出现在类型为 `class` 的 model 类中。而错误信息则是：
`Class 'xxx' has no initializers`
![](http://ww1.sinaimg.cn/large/b92f96b9ly1fq7k0frh76j20r902z3yr.jpg)

我们知道遵循了 `codable` 协议的 `class` 或者 `struct`, 编译器会自动添加相应的协议实现，即自动添加 `public init(from decoder: Decoder) throws` 和 `public func encode(to encoder: Encoder) throws` 的实现。

已经有了一个 `init(from decoder: Decoder)` 的 `initializer`了， 为什么还报出`Class 'xxx' has no initializers` 的错误呢？百思不得解😣😣


为了排除其他不相干的因素的影响，单独创建一个空白项目，创建一个类型为 `class` 的 `model`.

```swift
    class AdModel: Codable {
        var id: Int
        var link: String?
        var pic: String?
    }
```

和上图中的代码一毛一样，编译运行，Build Succeeded。没有报错😒，为什么？接着往下。

在 ViewController 的 viewDidLoad() 中 添加如下代码：

```swift
    let jsonString =
        """
        {
            "id": 1,
            "link": "http://www.google.com",
            "pic": null
        }
        """
    let data = jsonString.data(using: .utf8)!
        
    do {
        let admodel = try JSONDecoder().decode(AdModel.self, from: data)
        print(admodel)
    } catch {
        print(error)
    }
```

编译运行，Build Failed。报错😒😒，`Class 'AdModel' has no initializers`。

如果说手动添加两个协议方法的实现呢? 将下面的代码放到 AdModel 中：

```swift
    private enum CodingKeys: String, CodingKey {
        case id
        case link
        case pic
    }
    
    public required init(from decoder: Decoder) throws {
        let container = try  decoder.container(keyedBy: CodingKeys.self)
        id = try container.decode(Int.self, forKey: .id)
        link = try? container.decode(String.self, forKey: .link)
        pic = try? container.decode(String.self, forKey: .pic)
    }
    
    func encode(to encoder: Encoder) throws {
        var containder = encoder.container(keyedBy: CodingKeys.self)
        try containder.encode(id, forKey: .id)
        try containder.encode(link, forKey: .link)
        try containder.encode(pic, forKey: .pic)
    }
```

编译运行，Build Succeeded, 控制台输出解析后的 model。

到这里基本可以确定是因为什么了，猜测是因为编译器没有给 class 添加 codable 协议方法的默认实现。为了确定猜想是否正确，再 Google 一番,StackOverflow的一篇[帖子](https://stackoverflow.com/questions/49582558/codable-has-no-initializers-in-xcode-9-3-swift-4-1?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)情况与我的案例有点相似。在评论区找到一条评论说是 swift 的一个 [bug](https://bugs.swift.org/browse/SR-7315)。既然是个 bug，不可能等到苹果爸爸来修复。 

最简单同时最高效的解决办法就是将类型为 `class` 改为 `struct`, 由于将引用类型改为了值类型，所以代码中用到了为 `class` 类型的 `model` 的代码需要进行相应的修改。幸运的是类型是 `class` 的 `model` 一共就两个，改起来很简单，倘若多的话就悲剧了。

本想藉此机会再写一下关于 Codable 的一些东西，写来写去都是那些东西，网上重复的一大篇就不再重复了，贴几个写的比较好的帖子：
- [Ultimate Guide to JSON Parsing with Swift 4](https://benscheirman.com/2017/06/swift-json/)  
- [Swift 4 踩坑之 Codable 协议](https://www.jianshu.com/p/bdd9c012df15) 
- [Encoding and Decoding Custom Types](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)
- [Swift 4.0 Codable — Decoding subclasses, inherited classes, heterogeneous arrays](https://medium.com/tsengineering/swift-4-0-codable-decoding-subclasses-inherited-classes-heterogeneous-arrays-ee3e180eb556)