---
layout: post
title: Method Swizzle
subtitle: method-swizzle
date: 2018-11-01 10:56:13
categories: iOS
---


此文仅为记录性文章，由于老是忘记一些 API 的名称，故记录备忘

<!-- more -->


```
#import <Cocoa/Cocoa.h>

@interface ViewController : NSViewController

- (void)sendFeedback;
+ (void)uploadData;

- (void)playSong:(NSString *)song;


@end

#import "ViewController.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    // Do any additional setup after loading the view.
    
    [self sendFeedback];
    [[self class] uploadData];
    
    [self playSong:@"Struggle"];
}

- (void)setRepresentedObject:(id)representedObject {
    [super setRepresentedObject:representedObject];

    // Update the view, if already loaded.
}

- (void)playSong:(NSString *)song {
    NSLog(@"%@", song);
}

- (void)sendFeedback {
    NSLog(@"sendFeedback");
}

+ (void)uploadData {
    NSLog(@"uploadData");
}


@end

```

```
#import "ViewController.h"

NS_ASSUME_NONNULL_BEGIN

@interface ViewController (Swizzle)

@end

NS_ASSUME_NONNULL_END

#import "ViewController+Swizzle.h"
#import <objc/runtime.h>

/**
 * cls 是类对象时 swizzle 是实例方法，cls 是元类对象时 swizzle 的是类方法
 */
void swizzleMethod(Class cls, SEL originalSEL, SEL swizzleSEL) {
    Method originalMethod = class_getInstanceMethod(cls, originalSEL);
    Method swizzleMethod = class_getInstanceMethod(cls, swizzleSEL);
    
    BOOL didAdd = class_addMethod(cls, originalSEL, method_getImplementation(swizzleMethod), method_getTypeEncoding(swizzleMethod));
    if (didAdd) {
        class_replaceMethod(cls, swizzleSEL, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzleMethod);
    }
}

@implementation ViewController (Swizzle)

+ (void)load {
    // 此处是 swizzle 实例方法，cls 需要是类对象（实例方法存储在类对象中）
    // 此处的 self 指向类对象本身、[self class] 也只是简单返回 self
    swizzleMethod([self class], @selector(sendFeedback), @selector(xxx_sendFeedback));
    // 此处是 swizzle 类方法，cls 需要时元类对象（类方法存储在元类中）
    // 通过 object_getClass（）获取到元类对象
    swizzleMethod(object_getClass([self class]), @selector(uploadData), @selector(xxx_uploadData));
}

- (void)xxx_sendFeedback {
    NSLog(@"I'm swizzled");
//    [self xxx_sendFeedback];
}

+ (void)xxx_uploadData {
    NSLog(@"I'm swizzled too");
    [self xxx_uploadData];
}
```

