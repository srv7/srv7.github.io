---
layout: post
title: Realm+Object+Codable
date: 2018-08-09 10:54:12
subtitle: Realm+Object+Codable
categories: swift
published: false
---

想要将 App 中的 model 存入 Realm，那这个 model 则必须继承自 Realm 的 Object 类。而 Object 的对存储的属性类型具有一定的约束：

<!-- more -->

类型 | 非空值形式 | 可空值形式
---|---| --
Bool | `@objc dynamic var value = false` | `let value = RealmOptional<Bool>()`
Int | `@objc dynamic var value = 0` | `let value = RealmOptional<Int>()`
Float | `@objc dynamic var value: Float = 0.0` | `let value = RealmOptional<Float>()`
Double | `@objc dynamic var value: Double = 0.0` | `let value = RealmOptional<Double>()`
String | `@objc dynamic var value = "" ` | `@objc dynamic var value: String? = nil`
Data | `@objc dynamic var value = Data()` | `@objc dynamic var value: Data? = nil`
Date | `@objc dynamic var value = Date()` | `@objc dynamic var value: Date? = nil`
Object | 不存在：必须是可空值 |  `@objc dynamic var value: Class?`
List | `let value = List<Type>()`  |  `let value = List<Type>()`
LinkingObjects | `let value = LinkingObjects(fromType: Class.self, property: "property")` | 不存在：必须是非可空值

从上表可以看到，Bool、Int、Float、Double 的可选类型都不再是以前的 `Type?`形式了，而是采用了 `RealmOptional<Type>`的形式。数组也不再是 `[Type]`的形式而是使用Realm定义的 `List`。如此一来，这个 model 就对于 `Codable`就十分不友好了。

要想将此 model 用于 JSON 解析就有点不方便了，但依然手办法。以 `Book` 这个 Model 为例， 假设它有如下几个属性，`publishYear`, `name`, `isFinished`。(publishYear，isFinished 为可选类型)

```swift
@objcMembers
class Book: Object {
    dynamic var name: String = ""
    dynamic let publishYear = RealmOptional<Int>()
    dynamic let isFinished = RealmOptional<Bool>()
}
```

遵从 `Decodable`协议，并实现 `init(from decoder: Decoder)`


```swift
@objcMembers
class Book: Object, Decodable {
    dynamic var name: String = ""
    private(set) var publishYear = RealmOptional<Int>() // 此处一定要是 var 原因参见这里 https://gist.github.com/mishagray/3ee82a3a82f357bfbf8ff3b3d9eca5cd#gistcomment-2663165
    private(set) var isFinished = RealmOptional<Bool>() 
    
    private enum CodingKeys: String, CodingKey {
        case name
        case publishYear
        case isFinished
    }
    
    convenience required init(from decoder: Decoder) throws {
        self.init()
        let container =  try decoder.container(keyedBy: CodingKeys.self)
        name = try container.decode(String.self, forKey: .name)
        publishYear.value = try container.decodeIfPresent(Int.self, forKey: .publishYear)
        isFinished.value = try container.decodeIfPresent(Bool.self, forKey: .isFinished)
    }
}
```

测试一下：

```

let jsonData = """
[
  {
    "name": "Realm",
    "publishYear": 2018,
    "isFinished": true
  },
  {
    "name": "RxSwift",
    "publishYear": null,
    "isFinished": null
  }
]
"""

do {
    let books = try JSONDecoder().decode([Book].self, from: jsonData)
    print(books)
} catch {
    print(error)
}

```

输出：

```
[Book {
    name = Realm;
    publishYear = 2018;
    isFinished = 1;
}, Book {
    name = RxSwift;
    publishYear = (null);
    isFinished = (null);
}]
```
到这里就已经完成了 Object 用来解析 JSON 的工作。但依然有缺陷，就是用起来太麻烦。需要为每一个 继承自 Object 的 model 都手动进行解析。能不能再简化呢？答案是肯定的。 让 `RealmOptional` 和 `List` 实现 Codable 协议。


```swift
// swiftlint:disable line_length identifier_name
// stolen functions from the Swift stdlib
// https://github.com/apple/swift/blob/2e5817ebe15b8c2fc2459e08c1d462053cbb9a99/stdlib/public/core/Codable.swift
//
func assertTypeIsEncodable<T>(_ type: T.Type, in wrappingType: Any.Type) {
    guard T.self is Encodable.Type else {
        if T.self == Encodable.self || T.self == Codable.self {
            preconditionFailure("\(wrappingType) does not conform to Encodable because Encodable does not conform to itself. You must use a concrete type to encode or decode.")
        } else {
            preconditionFailure("\(wrappingType) does not conform to Encodable because \(T.self) does not conform to Encodable.")
        }
    }
}

func assertTypeIsDecodable<T>(_ type: T.Type, in wrappingType: Any.Type) {
    guard T.self is Decodable.Type else {
        if T.self == Decodable.self || T.self == Codable.self {
            preconditionFailure("\(wrappingType) does not conform to Decodable because Decodable does not conform to itself. You must use a concrete type to encode or decode.")
        } else {
            preconditionFailure("\(wrappingType) does not conform to Decodable because \(T.self) does not conform to Decodable.")
        }
    }
}

extension Encodable {
    func __encode(to container: inout SingleValueEncodingContainer) throws { try container.encode(self) }
    func __encode(to container: inout UnkeyedEncodingContainer)     throws { try container.encode(self) }
    func __encode<Key>(to container: inout KeyedEncodingContainer<Key>, forKey key: Key) throws { try container.encode(self, forKey: key) }
}

extension Decodable {
    // Since we cannot call these __init, we'll give the parameter a '__'.
    fileprivate init(__from container: SingleValueDecodingContainer)   throws { self = try container.decode(Self.self) }
    fileprivate init(__from container: inout UnkeyedDecodingContainer) throws { self = try container.decode(Self.self) }
    fileprivate init<Key>(__from container: KeyedDecodingContainer<Key>, forKey key: Key) throws { self = try container.decode(Self.self, forKey: key) }
}


extension RealmOptional : Encodable where Value : Encodable {
    public func encode(to encoder: Encoder) throws {
        assertTypeIsEncodable(Value.self, in: type(of: self))
        
        var container = encoder.singleValueContainer()
        if let v = self.value {
            try (v as Encodable).encode(to: encoder)  // swiftlint:disable:this force_cast
        } else {
            try container.encodeNil()
        }
    }
}

extension RealmOptional : Decodable where Value : Decodable {
    public convenience init(from decoder: Decoder) throws {
        // Initialize self here so we can get type(of: self).
        self.init()
        assertTypeIsDecodable(Value.self, in: type(of: self))
        
        let container = try decoder.singleValueContainer()
        if !container.decodeNil() {
            let metaType = (Value.self as Decodable.Type) // swiftlint:disable:this force_cast
            let element = try metaType.init(from: decoder)
            self.value = (element as! Value)  // swiftlint:disable:this force_cast
        }
    }
}
extension List : Decodable where Element : Decodable {
    public convenience init(from decoder: Decoder) throws {
        // Initialize self here so we can get type(of: self).
        self.init()
        assertTypeIsDecodable(Element.self, in: type(of: self))
        
        let metaType = (Element.self as Decodable.Type) // swiftlint:disable:this force_cast
        let container = try? decoder.unkeyedContainer()
        if var container = container {
            while !container.isAtEnd {
                let element = try metaType.init(__from: &container)
                self.append(element as! Element) // swiftlint:disable:this force_cast
            }
        } else {
            
        }
    }
}

extension List : Encodable where Element : Decodable {
    public func encode(to encoder: Encoder) throws {
        assertTypeIsEncodable(Element.self, in: type(of: self))
        
        var container = encoder.unkeyedContainer()
        for element in self {
            // superEncoder appends an empty element and wraps an Encoder around it.
            // This is normally appropriate for encoding super, but this is really what we want to do.
            let subencoder = container.superEncoder()
            try (element as! Encodable).encode(to: subencoder) // swiftlint:disable:this force_cast
        }
    }
}
```

上述代码来自[这里](https://gist.github.com/mishagray/3ee82a3a82f357bfbf8ff3b3d9eca5cd), 稍经修改。








