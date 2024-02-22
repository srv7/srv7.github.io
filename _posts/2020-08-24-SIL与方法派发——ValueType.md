---
layout: post
title: SIL与方法派发--值类型
date: 2020-08-24 14:41:47
subtitle: sil_method_dispatch_valueTypes
categories: swift
---
swift 版本
> Apple Swift version 5.2.4 (swiftlang-1103.0.32.9 clang-1103.0.32.53)

<!-- more -->

## SIL

### [function_ref](https://github.com/apple/swift/blob/master/docs/SIL.rst#function-ref)

> Creates a reference to a SIL function.

创建一个对 SIL 方法的引用，可以理解为一个指向方法实现的指针

### [dynamic-function-ref](https://github.com/apple/swift/blob/master/docs/SIL.rst#dynamic-function-ref)

> Creates a reference to a dynamically_replacable SIL function. A dynamically_replacable SIL function can be replaced at runtime.

创建一个对 `dynamically_replacable` SIL 方法的引用，可以理解为一个指向方法实现的指针。
`dynamically_replacable` 的 SIL 方法可以在运行时被替换。


## code
以 `struct` 为例，现有源代码 `struct.swift` 文件

```swift
import Foundation
struct Foo {
    func normalFunc() {}
    // @objc func objcFunc() {} // @objc can only be used with members of classes, @objc protocols, and concrete extensions of classes
    // final func finalFunc() {}  // error: only classes and class members may be marked with 'final'

    dynamic func dynamicFunc() {}
}

extension Foo {
    func extensionFunc() {}
}

let foo = Foo()
foo.normalFunc()
foo.extensionFunc()
foo.dynamicFunc()
```

将其编译为 sil

```shell
swiftc -emit-sil -Onone struct.swift > struct.swift.sil
```
得到 `struct.swift.sil` 文件

```swift
sil_stage canonical

import Builtin
import Swift
import SwiftShims

import Foundation

struct Foo {
  func normalFunc()
  dynamic func dynamicFunc()
  init()
}

extension Foo {
  func extensionFunc()
}

@_hasStorage @_hasInitialValue let foo: Foo { get }

// foo
sil_global hidden [let] @$s6struct3fooAA3FooVvp : $Foo

// main
sil @main : $@convention(c) (Int32, UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>) -> Int32 {
bb0(%0 : $Int32, %1 : $UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>):
  alloc_global @$s6struct3fooAA3FooVvp            // id: %2
  %3 = global_addr @$s6struct3fooAA3FooVvp : $*Foo // users: %14, %11, %8, %7
  %4 = metatype $@thin Foo.Type                   // user: %6
  // function_ref Foo.init()
  %5 = function_ref @$s6struct3FooVACycfC : $@convention(method) (@thin Foo.Type) -> Foo // user: %6
  %6 = apply %5(%4) : $@convention(method) (@thin Foo.Type) -> Foo // user: %7
  store %6 to %3 : $*Foo                          // id: %7
  %8 = load %3 : $*Foo                            // user: %10
  // function_ref Foo.normalFunc()
  %9 = function_ref @$s6struct3FooV10normalFuncyyF : $@convention(method) (Foo) -> () // user: %10
  %10 = apply %9(%8) : $@convention(method) (Foo) -> ()
  %11 = load %3 : $*Foo                           // user: %13
  // function_ref Foo.extensionFunc()
  %12 = function_ref @$s6struct3FooV13extensionFuncyyF : $@convention(method) (Foo) -> () // user: %13
  %13 = apply %12(%11) : $@convention(method) (Foo) -> ()
  %14 = load %3 : $*Foo                           // user: %16
  // dynamic_function_ref Foo.dynamicFunc()
  %15 = dynamic_function_ref @$s6struct3FooV11dynamicFuncyyF : $@convention(method) (Foo) -> () // user: %16
  %16 = apply %15(%14) : $@convention(method) (Foo) -> ()
  %17 = integer_literal $Builtin.Int32, 0         // user: %18
  %18 = struct $Int32 (%17 : $Builtin.Int32)      // user: %19
  return %18 : $Int32                             // id: %19
} // end sil function 'main'

// Foo.normalFunc()
sil hidden @$s6struct3FooV10normalFuncyyF : $@convention(method) (Foo) -> () {
// %0                                             // user: %1
bb0(%0 : $Foo):
  debug_value %0 : $Foo, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s6struct3FooV10normalFuncyyF'

// Foo.dynamicFunc()
sil hidden [dynamically_replacable] @$s6struct3FooV11dynamicFuncyyF : $@convention(method) (Foo) -> () {
// %0                                             // user: %1
bb0(%0 : $Foo):
  debug_value %0 : $Foo, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s6struct3FooV11dynamicFuncyyF'

// Foo.init()
sil hidden @$s6struct3FooVACycfC : $@convention(method) (@thin Foo.Type) -> Foo {
bb0(%0 : $@thin Foo.Type):
  %1 = alloc_stack $Foo, let, name "self"         // user: %3
  %2 = struct $Foo ()                             // user: %4
  dealloc_stack %1 : $*Foo                        // id: %3
  return %2 : $Foo                                // id: %4
} // end sil function '$s6struct3FooVACycfC'

// Foo.extensionFunc()
sil hidden @$s6struct3FooV13extensionFuncyyF : $@convention(method) (Foo) -> () {
// %0                                             // user: %1
bb0(%0 : $Foo):
  debug_value %0 : $Foo, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s6struct3FooV13extensionFuncyyF'
```

## 分析

|       方法         |          行号        |        指令       |        派发类型      |
| ---------------- | :-------------------: | :---------------: | :---------------: |
| **`func normalFunc()`** | 36 |       `function_ref`        |       `direct`      |
| **`dynamic func dynamicFunc()`** | 40 | `dynamic_function_ref` |       `direct`  |
| **`func extensionFunc()`** | 44  |       `function_ref`       |      `direct`    |

可以看出，对于值类型的原始方法声明、extension 中的方法声明，使用的都是静态调用的方式来进行方法派发的。

## 结论

|                    |   direct(直接派发)    | table(方法表派发) | message(消息派发) |
| :----------------: | :-------------------: | :---------------: | :---------------: |
| Value Type(值类型) | All Methods(所有方法) |        N/A        |        N/A        |
