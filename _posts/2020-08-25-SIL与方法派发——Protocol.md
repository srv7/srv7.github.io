---
layout: post
title: SIL与方法派发——Protocol
date: 2020-08-25 10:19:40
subtitle: sil_method_dispatch_protocol
categories: swift
---
swift 版本
> Apple Swift version 5.2.4 (swiftlang-1103.0.32.9 clang-1103.0.32.53)

<!-- more -->

## SIL

### [function_ref](https://github.com/apple/swift/blob/master/docs/SIL.rst#function-ref)

> Creates a reference to a SIL function.

创建一个对 SIL 方法的引用，可以理解为一个指向方法实现的指针

### [class-method](https://github.com/apple/swift/blob/master/docs/SIL.rst#class-method)

> The `class_method` and `super_method` instructions must reference Swift native methods and always use vtable dispatch.


`class_method` 和 `super_method` 指令必须引用 swift 原生方法，并永远使用 `vtable`(方发表)派发

### [objc_method](https://github.com/apple/swift/blob/master/docs/SIL.rst#objc-method)

> The `objc_method` and `bjc_super_method` instructions must reference Objective-C methods (indicated by the foreign marker on a method reference, as in #NSObject.description!foreign).

`objc_method` 和 `bjc_super_method` 指令必须引用 Objective-C 方法(在方法引用上使用 `foreign` 标记来标识)

### [witness_method](https://github.com/apple/swift/blob/master/docs/SIL.rst#witness-method)

> Looks up the implementation of a protocol method for a generic type variable constrained by that protocol. The result will be generic on the Self archetype of the original protocol and have the witness_method calling convention. If the referenced protocol is an `@objc` protocol, the resulting type has the objc calling convention.


## code

现有源代码 `protocol.swift` 文件

```swift
import Foundation
protocol FooProtocol {
    func initialProtocolMethod()
    func initialProtocolMethodWithExtension()
}

extension FooProtocol {
    func initialProtocolMethodWithExtension() {}
    func protocolExtensionMethod() {}
    func protocolExtensionMethodConfirmingImp() {}
}

struct BarStruct: FooProtocol {
    func initialStructMethod() {}
    func initialProtocolMethod() {}
    func initialProtocolMethodWithExtension() {}
    func protocolExtensionMethodConfirmingImp() {}
}

class BazClass: FooProtocol {
    func initialClassMethod() {}
    func initialProtocolMethod() {}
    func initialProtocolMethodWithExtension() {}
    func protocolExtensionMethodConfirmingImp() {}
}

@objc protocol ObjcProtocol {
    func objcInitialProtocolMethod()
}

extension BazClass: ObjcProtocol {
    func objcInitialProtocolMethod() {}
}

let bar: BarStruct = BarStruct()
bar.initialStructMethod()
bar.initialProtocolMethod()
bar.initialProtocolMethodWithExtension()
bar.protocolExtensionMethod()
bar.protocolExtensionMethodConfirmingImp()

let baz: BazClass = BazClass()
baz.initialClassMethod()
baz.initialProtocolMethod()
baz.initialProtocolMethodWithExtension()
baz.protocolExtensionMethod()
baz.protocolExtensionMethodConfirmingImp()
baz.objcInitialProtocolMethod()

let bzz: FooProtocol = BazClass()
bzz.initialProtocolMethod()
bzz.initialProtocolMethodWithExtension()
bzz.protocolExtensionMethod()
bzz.protocolExtensionMethodConfirmingImp()

let foo: FooProtocol = BarStruct()
foo.initialProtocolMethod()
foo.initialProtocolMethodWithExtension()
foo.protocolExtensionMethod()
foo.protocolExtensionMethodConfirmingImp()
```

将其编译为 sil

```shell
swiftc -emit-sil -Onone protocol.swift > protocol.swift.sil
```
得到 `protocol.swift.sil` 文件

```swift
sil_stage canonical

import Builtin
import Swift
import SwiftShims

import Foundation

protocol FooProtocol {
  func initialProtocolMethod()
  func initialProtocolMethodWithExtension()
}

extension FooProtocol {
  func initialProtocolMethodWithExtension()
  func protocolExtensionMethod()
  func protocolExtensionMethodConfirmingImp()
}

struct BarStruct : FooProtocol {
  func initialProtocolMethod()
  func initialProtocolMethodWithExtension()
  func protocolExtensionMethodConfirmingImp()
  init()
}

class BazClass : FooProtocol {
  func initialProtocolMethod()
  func initialProtocolMethodWithExtension()
  func protocolExtensionMethodConfirmingImp()
  @objc deinit
  init()
}

@objc protocol ObjcProtocol {
  @objc func objcInitialProtocolMethod()
}

extension BazClass : ObjcProtocol {
  dynamic func objcInitialProtocolMethod()
}

@_hasStorage @_hasInitialValue let bar: BarStruct { get }

@_hasStorage @_hasInitialValue let baz: BazClass { get }

@_hasStorage @_hasInitialValue let bzz: FooProtocol { get }

@_hasStorage @_hasInitialValue let foo: FooProtocol { get }

// bar
sil_global hidden [let] @$s8protocol3barAA9BarStructVvp : $BarStruct

// baz
sil_global hidden [let] @$s8protocol3bazAA8BazClassCvp : $BazClass

// bzz
sil_global hidden [let] @$s8protocol3bzzAA11FooProtocol_pvp : $FooProtocol

// foo
sil_global hidden [let] @$s8protocol3fooAA11FooProtocol_pvp : $FooProtocol

// main
sil @main : $@convention(c) (Int32, UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>) -> Int32 {
bb0(%0 : $Int32, %1 : $UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>):
  alloc_global @$s8protocol3barAA9BarStructVvp    // id: %2
  %3 = global_addr @$s8protocol3barAA9BarStructVvp : $*BarStruct // users: %20, %14, %11, %8, %7
  %4 = metatype $@thin BarStruct.Type             // user: %6
  // function_ref BarStruct.init()
  %5 = function_ref @$s8protocol9BarStructVACycfC : $@convention(method) (@thin BarStruct.Type) -> BarStruct // user: %6
  %6 = apply %5(%4) : $@convention(method) (@thin BarStruct.Type) -> BarStruct // user: %7
  store %6 to %3 : $*BarStruct                    // id: %7
  %8 = load %3 : $*BarStruct                      // user: %10
  // function_ref BarStruct.initialProtocolMethod()
  %9 = function_ref @$s8protocol9BarStructV21initialProtocolMethodyyF : $@convention(method) (BarStruct) -> () // user: %10
  %10 = apply %9(%8) : $@convention(method) (BarStruct) -> ()
  %11 = load %3 : $*BarStruct                     // user: %13
  // function_ref BarStruct.initialProtocolMethodWithExtension()
  %12 = function_ref @$s8protocol9BarStructV34initialProtocolMethodWithExtensionyyF : $@convention(method) (BarStruct) -> () // user: %13
  %13 = apply %12(%11) : $@convention(method) (BarStruct) -> ()
  %14 = load %3 : $*BarStruct                     // user: %16
  %15 = alloc_stack $BarStruct                    // users: %16, %19, %18
  store %14 to %15 : $*BarStruct                  // id: %16
  // function_ref FooProtocol.protocolExtensionMethod()
  %17 = function_ref @$s8protocol11FooProtocolPAAE0A15ExtensionMethodyyF : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // user: %18
  %18 = apply %17<BarStruct>(%15) : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> ()
  dealloc_stack %15 : $*BarStruct                 // id: %19
  %20 = load %3 : $*BarStruct                     // user: %22
  // function_ref BarStruct.protocolExtensionMethodConfirmingImp()
  %21 = function_ref @$s8protocol9BarStructV0A28ExtensionMethodConfirmingImpyyF : $@convention(method) (BarStruct) -> () // user: %22
  %22 = apply %21(%20) : $@convention(method) (BarStruct) -> ()
  alloc_global @$s8protocol3bazAA8BazClassCvp     // id: %23
  %24 = global_addr @$s8protocol3bazAA8BazClassCvp : $*BazClass // users: %44, %41, %35, %32, %29, %28
  %25 = metatype $@thick BazClass.Type            // user: %27
  // function_ref BazClass.__allocating_init()
  %26 = function_ref @$s8protocol8BazClassCACycfC : $@convention(method) (@thick BazClass.Type) -> @owned BazClass // user: %27
  %27 = apply %26(%25) : $@convention(method) (@thick BazClass.Type) -> @owned BazClass // user: %28
  store %27 to %24 : $*BazClass                   // id: %28
  %29 = load %24 : $*BazClass                     // users: %30, %31
  %30 = class_method %29 : $BazClass, #BazClass.initialProtocolMethod!1 : (BazClass) -> () -> (), $@convention(method) (@guaranteed BazClass) -> () // user: %31
  %31 = apply %30(%29) : $@convention(method) (@guaranteed BazClass) -> ()
  %32 = load %24 : $*BazClass                     // users: %33, %34
  %33 = class_method %32 : $BazClass, #BazClass.initialProtocolMethodWithExtension!1 : (BazClass) -> () -> (), $@convention(method) (@guaranteed BazClass) -> () // user: %34
  %34 = apply %33(%32) : $@convention(method) (@guaranteed BazClass) -> ()
  %35 = load %24 : $*BazClass                     // user: %37
  %36 = alloc_stack $BazClass                     // users: %37, %40, %39
  store %35 to %36 : $*BazClass                   // id: %37
  // function_ref FooProtocol.protocolExtensionMethod()
  %38 = function_ref @$s8protocol11FooProtocolPAAE0A15ExtensionMethodyyF : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // user: %39
  %39 = apply %38<BazClass>(%36) : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> ()
  dealloc_stack %36 : $*BazClass                  // id: %40
  %41 = load %24 : $*BazClass                     // users: %42, %43
  %42 = class_method %41 : $BazClass, #BazClass.protocolExtensionMethodConfirmingImp!1 : (BazClass) -> () -> (), $@convention(method) (@guaranteed BazClass) -> () // user: %43
  %43 = apply %42(%41) : $@convention(method) (@guaranteed BazClass) -> ()
  %44 = load %24 : $*BazClass                     // users: %45, %46
  %45 = objc_method %44 : $BazClass, #BazClass.objcInitialProtocolMethod!1.foreign : (BazClass) -> () -> (), $@convention(objc_method) (BazClass) -> () // user: %46
  %46 = apply %45(%44) : $@convention(objc_method) (BazClass) -> ()
  alloc_global @$s8protocol3bzzAA11FooProtocol_pvp // id: %47
  %48 = global_addr @$s8protocol3bzzAA11FooProtocol_pvp : $*FooProtocol // users: %63, %60, %57, %54, %52
  %49 = metatype $@thick BazClass.Type            // user: %51
  // function_ref BazClass.__allocating_init()
  %50 = function_ref @$s8protocol8BazClassCACycfC : $@convention(method) (@thick BazClass.Type) -> @owned BazClass // user: %51
  %51 = apply %50(%49) : $@convention(method) (@thick BazClass.Type) -> @owned BazClass // user: %53
  %52 = init_existential_addr %48 : $*FooProtocol, $BazClass // user: %53
  store %51 to %52 : $*BazClass                   // id: %53
  %54 = open_existential_addr immutable_access %48 : $*FooProtocol to $*@opened("3160EC6E-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %56, %56, %55
  %55 = witness_method $@opened("3160EC6E-E686-11EA-8CE2-ACDE48001122") FooProtocol, #FooProtocol.initialProtocolMethod!1 : <Self where Self : FooProtocol> (Self) -> () -> (), %54 : $*@opened("3160EC6E-E686-11EA-8CE2-ACDE48001122") FooProtocol : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %54; user: %56
  %56 = apply %55<@opened("3160EC6E-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%54) : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %54
  %57 = open_existential_addr immutable_access %48 : $*FooProtocol to $*@opened("3160EF3E-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %59, %59, %58
  %58 = witness_method $@opened("3160EF3E-E686-11EA-8CE2-ACDE48001122") FooProtocol, #FooProtocol.initialProtocolMethodWithExtension!1 : <Self where Self : FooProtocol> (Self) -> () -> (), %57 : $*@opened("3160EF3E-E686-11EA-8CE2-ACDE48001122") FooProtocol : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %57; user: %59
  %59 = apply %58<@opened("3160EF3E-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%57) : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %57
  %60 = open_existential_addr immutable_access %48 : $*FooProtocol to $*@opened("3160F164-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %62, %62
  // function_ref FooProtocol.protocolExtensionMethod()
  %61 = function_ref @$s8protocol11FooProtocolPAAE0A15ExtensionMethodyyF : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // user: %62
  %62 = apply %61<@opened("3160F164-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%60) : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %60
  %63 = open_existential_addr immutable_access %48 : $*FooProtocol to $*@opened("3160F2D6-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %65, %65
  // function_ref FooProtocol.protocolExtensionMethodConfirmingImp()
  %64 = function_ref @$s8protocol11FooProtocolPAAE0A28ExtensionMethodConfirmingImpyyF : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // user: %65
  %65 = apply %64<@opened("3160F2D6-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%63) : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %63
  alloc_global @$s8protocol3fooAA11FooProtocol_pvp // id: %66
  %67 = global_addr @$s8protocol3fooAA11FooProtocol_pvp : $*FooProtocol // users: %82, %79, %76, %73, %71
  %68 = metatype $@thin BarStruct.Type            // user: %70
  // function_ref BarStruct.init()
  %69 = function_ref @$s8protocol9BarStructVACycfC : $@convention(method) (@thin BarStruct.Type) -> BarStruct // user: %70
  %70 = apply %69(%68) : $@convention(method) (@thin BarStruct.Type) -> BarStruct // user: %72
  %71 = init_existential_addr %67 : $*FooProtocol, $BarStruct // user: %72
  store %70 to %71 : $*BarStruct                  // id: %72
  %73 = open_existential_addr immutable_access %67 : $*FooProtocol to $*@opened("3160F704-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %75, %75, %74
  %74 = witness_method $@opened("3160F704-E686-11EA-8CE2-ACDE48001122") FooProtocol, #FooProtocol.initialProtocolMethod!1 : <Self where Self : FooProtocol> (Self) -> () -> (), %73 : $*@opened("3160F704-E686-11EA-8CE2-ACDE48001122") FooProtocol : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %73; user: %75
  %75 = apply %74<@opened("3160F704-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%73) : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %73
  %76 = open_existential_addr immutable_access %67 : $*FooProtocol to $*@opened("3160F8EE-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %78, %78, %77
  %77 = witness_method $@opened("3160F8EE-E686-11EA-8CE2-ACDE48001122") FooProtocol, #FooProtocol.initialProtocolMethodWithExtension!1 : <Self where Self : FooProtocol> (Self) -> () -> (), %76 : $*@opened("3160F8EE-E686-11EA-8CE2-ACDE48001122") FooProtocol : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %76; user: %78
  %78 = apply %77<@opened("3160F8EE-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%76) : $@convention(witness_method: FooProtocol) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %76
  %79 = open_existential_addr immutable_access %67 : $*FooProtocol to $*@opened("3160FBAA-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %81, %81
  // function_ref FooProtocol.protocolExtensionMethod()
  %80 = function_ref @$s8protocol11FooProtocolPAAE0A15ExtensionMethodyyF : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // user: %81
  %81 = apply %80<@opened("3160FBAA-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%79) : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %79
  %82 = open_existential_addr immutable_access %67 : $*FooProtocol to $*@opened("3160FEE8-E686-11EA-8CE2-ACDE48001122") FooProtocol // users: %84, %84
  // function_ref FooProtocol.protocolExtensionMethodConfirmingImp()
  %83 = function_ref @$s8protocol11FooProtocolPAAE0A28ExtensionMethodConfirmingImpyyF : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // user: %84
  %84 = apply %83<@opened("3160FEE8-E686-11EA-8CE2-ACDE48001122") FooProtocol>(%82) : $@convention(method) <τ_0_0 where τ_0_0 : FooProtocol> (@in_guaranteed τ_0_0) -> () // type-defs: %82
  %85 = integer_literal $Builtin.Int32, 0         // user: %86
  %86 = struct $Int32 (%85 : $Builtin.Int32)      // user: %87
  return %86 : $Int32                             // id: %87
} // end sil function 'main'

// FooProtocol.initialProtocolMethodWithExtension()
sil hidden @$s8protocol11FooProtocolPAAE07initialC19MethodWithExtensionyyF : $@convention(method) <Self where Self : FooProtocol> (@in_guaranteed Self) -> () {
// %0                                             // user: %1
bb0(%0 : $*Self):
  debug_value_addr %0 : $*Self, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol11FooProtocolPAAE07initialC19MethodWithExtensionyyF'

// FooProtocol.protocolExtensionMethod()
sil hidden @$s8protocol11FooProtocolPAAE0A15ExtensionMethodyyF : $@convention(method) <Self where Self : FooProtocol> (@in_guaranteed Self) -> () {
// %0                                             // user: %1
bb0(%0 : $*Self):
  debug_value_addr %0 : $*Self, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol11FooProtocolPAAE0A15ExtensionMethodyyF'

// FooProtocol.protocolExtensionMethodConfirmingImp()
sil hidden @$s8protocol11FooProtocolPAAE0A28ExtensionMethodConfirmingImpyyF : $@convention(method) <Self where Self : FooProtocol> (@in_guaranteed Self) -> () {
// %0                                             // user: %1
bb0(%0 : $*Self):
  debug_value_addr %0 : $*Self, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol11FooProtocolPAAE0A28ExtensionMethodConfirmingImpyyF'

// BarStruct.initialProtocolMethod()
sil hidden @$s8protocol9BarStructV21initialProtocolMethodyyF : $@convention(method) (BarStruct) -> () {
// %0                                             // user: %1
bb0(%0 : $BarStruct):
  debug_value %0 : $BarStruct, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol9BarStructV21initialProtocolMethodyyF'

// BarStruct.initialProtocolMethodWithExtension()
sil hidden @$s8protocol9BarStructV34initialProtocolMethodWithExtensionyyF : $@convention(method) (BarStruct) -> () {
// %0                                             // user: %1
bb0(%0 : $BarStruct):
  debug_value %0 : $BarStruct, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol9BarStructV34initialProtocolMethodWithExtensionyyF'

// BarStruct.protocolExtensionMethodConfirmingImp()
sil hidden @$s8protocol9BarStructV0A28ExtensionMethodConfirmingImpyyF : $@convention(method) (BarStruct) -> () {
// %0                                             // user: %1
bb0(%0 : $BarStruct):
  debug_value %0 : $BarStruct, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol9BarStructV0A28ExtensionMethodConfirmingImpyyF'

// BarStruct.init()
sil hidden @$s8protocol9BarStructVACycfC : $@convention(method) (@thin BarStruct.Type) -> BarStruct {
bb0(%0 : $@thin BarStruct.Type):
  %1 = alloc_stack $BarStruct, let, name "self"   // user: %3
  %2 = struct $BarStruct ()                       // user: %4
  dealloc_stack %1 : $*BarStruct                  // id: %3
  return %2 : $BarStruct                          // id: %4
} // end sil function '$s8protocol9BarStructVACycfC'

// protocol witness for FooProtocol.initialProtocolMethod() in conformance BarStruct
sil private [transparent] [thunk] @$s8protocol9BarStructVAA11FooProtocolA2aDP07initialE6MethodyyFTW : $@convention(witness_method: FooProtocol) (@in_guaranteed BarStruct) -> () {
// %0                                             // user: %1
bb0(%0 : $*BarStruct):
  %1 = load %0 : $*BarStruct                      // user: %3
  // function_ref BarStruct.initialProtocolMethod()
  %2 = function_ref @$s8protocol9BarStructV21initialProtocolMethodyyF : $@convention(method) (BarStruct) -> () // user: %3
  %3 = apply %2(%1) : $@convention(method) (BarStruct) -> ()
  %4 = tuple ()                                   // user: %5
  return %4 : $()                                 // id: %5
} // end sil function '$s8protocol9BarStructVAA11FooProtocolA2aDP07initialE6MethodyyFTW'

// protocol witness for FooProtocol.initialProtocolMethodWithExtension() in conformance BarStruct
sil private [transparent] [thunk] @$s8protocol9BarStructVAA11FooProtocolA2aDP07initialE19MethodWithExtensionyyFTW : $@convention(witness_method: FooProtocol) (@in_guaranteed BarStruct) -> () {
// %0                                             // user: %1
bb0(%0 : $*BarStruct):
  %1 = load %0 : $*BarStruct                      // user: %3
  // function_ref BarStruct.initialProtocolMethodWithExtension()
  %2 = function_ref @$s8protocol9BarStructV34initialProtocolMethodWithExtensionyyF : $@convention(method) (BarStruct) -> () // user: %3
  %3 = apply %2(%1) : $@convention(method) (BarStruct) -> ()
  %4 = tuple ()                                   // user: %5
  return %4 : $()                                 // id: %5
} // end sil function '$s8protocol9BarStructVAA11FooProtocolA2aDP07initialE19MethodWithExtensionyyFTW'

// BazClass.initialProtocolMethod()
sil hidden @$s8protocol8BazClassC21initialProtocolMethodyyF : $@convention(method) (@guaranteed BazClass) -> () {
// %0                                             // user: %1
bb0(%0 : $BazClass):
  debug_value %0 : $BazClass, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol8BazClassC21initialProtocolMethodyyF'

// BazClass.initialProtocolMethodWithExtension()
sil hidden @$s8protocol8BazClassC34initialProtocolMethodWithExtensionyyF : $@convention(method) (@guaranteed BazClass) -> () {
// %0                                             // user: %1
bb0(%0 : $BazClass):
  debug_value %0 : $BazClass, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol8BazClassC34initialProtocolMethodWithExtensionyyF'

// BazClass.protocolExtensionMethodConfirmingImp()
sil hidden @$s8protocol8BazClassC0A28ExtensionMethodConfirmingImpyyF : $@convention(method) (@guaranteed BazClass) -> () {
// %0                                             // user: %1
bb0(%0 : $BazClass):
  debug_value %0 : $BazClass, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol8BazClassC0A28ExtensionMethodConfirmingImpyyF'

// BazClass.deinit
sil hidden @$s8protocol8BazClassCfd : $@convention(method) (@guaranteed BazClass) -> @owned Builtin.NativeObject {
// %0                                             // users: %2, %1
bb0(%0 : $BazClass):
  debug_value %0 : $BazClass, let, name "self", argno 1 // id: %1
  %2 = unchecked_ref_cast %0 : $BazClass to $Builtin.NativeObject // user: %3
  return %2 : $Builtin.NativeObject               // id: %3
} // end sil function '$s8protocol8BazClassCfd'

// BazClass.__deallocating_deinit
sil hidden @$s8protocol8BazClassCfD : $@convention(method) (@owned BazClass) -> () {
// %0                                             // users: %3, %1
bb0(%0 : $BazClass):
  debug_value %0 : $BazClass, let, name "self", argno 1 // id: %1
  // function_ref BazClass.deinit
  %2 = function_ref @$s8protocol8BazClassCfd : $@convention(method) (@guaranteed BazClass) -> @owned Builtin.NativeObject // user: %3
  %3 = apply %2(%0) : $@convention(method) (@guaranteed BazClass) -> @owned Builtin.NativeObject // user: %4
  %4 = unchecked_ref_cast %3 : $Builtin.NativeObject to $BazClass // user: %5
  dealloc_ref %4 : $BazClass                      // id: %5
  %6 = tuple ()                                   // user: %7
  return %6 : $()                                 // id: %7
} // end sil function '$s8protocol8BazClassCfD'

// BazClass.__allocating_init()
sil hidden [exact_self_class] @$s8protocol8BazClassCACycfC : $@convention(method) (@thick BazClass.Type) -> @owned BazClass {
bb0(%0 : $@thick BazClass.Type):
  %1 = alloc_ref $BazClass                        // user: %3
  // function_ref BazClass.init()
  %2 = function_ref @$s8protocol8BazClassCACycfc : $@convention(method) (@owned BazClass) -> @owned BazClass // user: %3
  %3 = apply %2(%1) : $@convention(method) (@owned BazClass) -> @owned BazClass // user: %4
  return %3 : $BazClass                           // id: %4
} // end sil function '$s8protocol8BazClassCACycfC'

// BazClass.init()
sil hidden @$s8protocol8BazClassCACycfc : $@convention(method) (@owned BazClass) -> @owned BazClass {
// %0                                             // users: %2, %1
bb0(%0 : $BazClass):
  debug_value %0 : $BazClass, let, name "self", argno 1 // id: %1
  return %0 : $BazClass                           // id: %2
} // end sil function '$s8protocol8BazClassCACycfc'

// protocol witness for FooProtocol.initialProtocolMethod() in conformance BazClass
sil private [transparent] [thunk] @$s8protocol8BazClassCAA11FooProtocolA2aDP07initialE6MethodyyFTW : $@convention(witness_method: FooProtocol) (@in_guaranteed BazClass) -> () {
// %0                                             // user: %1
bb0(%0 : $*BazClass):
  %1 = load %0 : $*BazClass                       // users: %2, %3
  %2 = class_method %1 : $BazClass, #BazClass.initialProtocolMethod!1 : (BazClass) -> () -> (), $@convention(method) (@guaranteed BazClass) -> () // user: %3
  %3 = apply %2(%1) : $@convention(method) (@guaranteed BazClass) -> ()
  %4 = tuple ()                                   // user: %5
  return %4 : $()                                 // id: %5
} // end sil function '$s8protocol8BazClassCAA11FooProtocolA2aDP07initialE6MethodyyFTW'

// protocol witness for FooProtocol.initialProtocolMethodWithExtension() in conformance BazClass
sil private [transparent] [thunk] @$s8protocol8BazClassCAA11FooProtocolA2aDP07initialE19MethodWithExtensionyyFTW : $@convention(witness_method: FooProtocol) (@in_guaranteed BazClass) -> () {
// %0                                             // user: %1
bb0(%0 : $*BazClass):
  %1 = load %0 : $*BazClass                       // users: %2, %3
  %2 = class_method %1 : $BazClass, #BazClass.initialProtocolMethodWithExtension!1 : (BazClass) -> () -> (), $@convention(method) (@guaranteed BazClass) -> () // user: %3
  %3 = apply %2(%1) : $@convention(method) (@guaranteed BazClass) -> ()
  %4 = tuple ()                                   // user: %5
  return %4 : $()                                 // id: %5
} // end sil function '$s8protocol8BazClassCAA11FooProtocolA2aDP07initialE19MethodWithExtensionyyFTW'

// BazClass.objcInitialProtocolMethod()
sil hidden @$s8protocol8BazClassC25objcInitialProtocolMethodyyF : $@convention(method) (@guaranteed BazClass) -> () {
// %0                                             // user: %1
bb0(%0 : $BazClass):
  debug_value %0 : $BazClass, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s8protocol8BazClassC25objcInitialProtocolMethodyyF'

// @objc BazClass.objcInitialProtocolMethod()
sil hidden [thunk] @$s8protocol8BazClassC25objcInitialProtocolMethodyyFTo : $@convention(objc_method) (BazClass) -> () {
// %0                                             // users: %4, %3, %1
bb0(%0 : $BazClass):
  strong_retain %0 : $BazClass                    // id: %1
  // function_ref BazClass.objcInitialProtocolMethod()
  %2 = function_ref @$s8protocol8BazClassC25objcInitialProtocolMethodyyF : $@convention(method) (@guaranteed BazClass) -> () // user: %3
  %3 = apply %2(%0) : $@convention(method) (@guaranteed BazClass) -> () // user: %5
  strong_release %0 : $BazClass                   // id: %4
  return %3 : $()                                 // id: %5
} // end sil function '$s8protocol8BazClassC25objcInitialProtocolMethodyyFTo'

sil_vtable BazClass {
  #BazClass.initialProtocolMethod!1: (BazClass) -> () -> () : @$s8protocol8BazClassC21initialProtocolMethodyyF	// BazClass.initialProtocolMethod()
  #BazClass.initialProtocolMethodWithExtension!1: (BazClass) -> () -> () : @$s8protocol8BazClassC34initialProtocolMethodWithExtensionyyF	// BazClass.initialProtocolMethodWithExtension()
  #BazClass.protocolExtensionMethodConfirmingImp!1: (BazClass) -> () -> () : @$s8protocol8BazClassC0A28ExtensionMethodConfirmingImpyyF	// BazClass.protocolExtensionMethodConfirmingImp()
  #BazClass.init!allocator.1: (BazClass.Type) -> () -> BazClass : @$s8protocol8BazClassCACycfC	// BazClass.__allocating_init()
  #BazClass.deinit!deallocator.1: @$s8protocol8BazClassCfD	// BazClass.__deallocating_deinit
}

sil_witness_table hidden BarStruct: FooProtocol module protocol {
  method #FooProtocol.initialProtocolMethod!1: <Self where Self : FooProtocol> (Self) -> () -> () : @$s8protocol9BarStructVAA11FooProtocolA2aDP07initialE6MethodyyFTW	// protocol witness for FooProtocol.initialProtocolMethod() in conformance BarStruct
  method #FooProtocol.initialProtocolMethodWithExtension!1: <Self where Self : FooProtocol> (Self) -> () -> () : @$s8protocol9BarStructVAA11FooProtocolA2aDP07initialE19MethodWithExtensionyyFTW	// protocol witness for FooProtocol.initialProtocolMethodWithExtension() in conformance BarStruct
}

sil_witness_table hidden BazClass: FooProtocol module protocol {
  method #FooProtocol.initialProtocolMethod!1: <Self where Self : FooProtocol> (Self) -> () -> () : @$s8protocol8BazClassCAA11FooProtocolA2aDP07initialE6MethodyyFTW	// protocol witness for FooProtocol.initialProtocolMethod() in conformance BazClass
  method #FooProtocol.initialProtocolMethodWithExtension!1: <Self where Self : FooProtocol> (Self) -> () -> () : @$s8protocol8BazClassCAA11FooProtocolA2aDP07initialE19MethodWithExtensionyyFTW	// protocol witness for FooProtocol.initialProtocolMethodWithExtension() in conformance BazClass
}
```

## 分析

### struct 实现协议

|                    |          行号        |        指令       |        派发类型     |
| ---------------- | :-------------------: | :---------------: | :---------------: |
| **`func initialProtocolMethod()`** |  75 |   `function_ref`  |      `Direct`     |
| **`func initialProtocolMethodWithExtension()`** |  79 |  `function_ref` | `Direct`  |
| **`func protocolExtensionMethod()`** |  85 |   `function_ref`  | `Direct`  |
| **`func protocolExtensionMethodConfirmingImp()`** |  90 |   `function_ref`  | `Direct`  |

**如果遵循协议的是 struct，则所有方法都是静态调用**

### class 实现协议
|                    |          行号        |        指令       |        派发类型     |
| ---------------- | :-------------------: | :---------------: | :---------------: |
| **`func initialProtocolMethod()`** |  100 |   `class_method`    |      `table`       |
| **`func initialProtocolMethodWithExtension()`** |  103 |   `class_method`  | `table`  |
| **`func protocolExtensionMethod()`** |  109 |   `function_ref`  | `Direct`  |
| **`func protocolExtensionMethodConfirmingImp()`** |  113 |   `class_method`  | `table`  |

**如果遵循协议的是 class, class 实现的协议方法都是通过方发表派发，protocol extension 中声明但 class 未实现的方法使用的是静态调用**

**Note:** 此处所说的 class 遵循协议是指在 声明 class 时就添加协议声明（`class BazClass: FooProtocol`），通过 extension 遵循协议，其派发规则与在 class extension 中声明的方法一致

### 变量类型是协议类型

如 `let bzz: FooProtocol = BazClass()` 或 `let foo: FooProtocol = BarStruct()`

在协议声明范围内声明的方法存在于目击表中，派发方式是方发表派发

而在extension 中声明的方法不存在与目击表中，派发方式是静态调用

|       方法             |          行号        |        指令       |        派发类型     |
| ---------------- | :-------------------: | :---------------: | :---------------: |
| **`func initialProtocolMethod()`** |  127/149 |   `witness_method`    |      `vtable`       |
| **`func initialProtocolMethodWithExtension()`** |  130/152 |   `witness_method`  | `vtable`  |
| **`func protocolExtensionMethod()`** |  134/156 |   `function_ref`  | `direct`  |
| **`func protocolExtensionMethodConfirmingImp()`** |  138/160 |   `function_ref`  | `direct`  |

### 协议有 @objc

```swift
@objc protocol ObjcProtocol {
    func objcInitialProtocolMethod()
}
```
|       方法             |          行号        |        指令       |        派发类型     |
| ---------------- | :-------------------: | :---------------: | :---------------: |
| **`func objcInitialProtocolMethod()`** |  116 |   `objc_method`    |      `messsage`  |

**@objc 修饰的protocol的方法派发方式是消息派发**


## 结论

|        方法            |   Direct(直接派发)    | Table(方法表派发) | Message(消息派发) |
| :----------------: | :-------------------: | :---------------: | :---------------: |
| protocol | 1. extension 中声明的方法 2. `struct` 实现的协议方法 3. class 通过 extension 遵循协议的方法  |   class 声明范围内遵循的协议方法  | 带有 `@objc` 协议的方法 |