---
layout: post
title: framework与vendored_framework
date: 2021-05-19 13:59:43
categories: iOS
tags:
    - cocoapods
---
# framework 与 vendored_framework

本文记录了使用 xcode 创建 Framework 的流程、使用 cocoapods 使用已有 Framework 的流程。

<!--more-->

## 创建 Static Framework 或 Library

- 创建 `framework / `, `xcode -> File -> New -> Project -> iOS -> Framework/Static Library`, 如果是 `Framework` 修改 `build settings` 的 `Mach-O Type` 为 `Static Library`。修改 `build libraries for distribution` 为 YES。
- 创建 资源文件 `Target`, `File -> New -> Target -> MacOS -> Bundle`, 修改 `BaseSDK` 为 `iOS`。修改 `COMBINE_HIDPI_IMAGES` 为 NO。
- 创建 Demo 工程, `File -> New -> Target -> iOS -> App`。
- 设置 Target 依赖。
    - 选中 Demo 的 `Target -> Build Phases -> Dependencies -> + -> 选中 framework/library` 并添加
    - 选中 `framework/library` 的` Target -> Build Phases -> Dependencies -> + -> 选中 bundle` 并添加
- 创建源文件，`Target Membership` 选择 `framework/library` 的 Target。并正确设置 头文件在 public、private 和 project 的位置，并正确的在 framework 的 头文件中 import 。
- 添加资源文件， `Target Membership` 选择 bundle 的 Target。
> https://09mejohn.medium.com/resource-bundles-in-ios-static-library-beba3070fafd

### 手动集成到项目
选中要添加 framework 的 `group -> add files to project -> copy if needed、create groups、 add to targets`。在 target 的 `General ->Frameworks,Libraries,and Embedded Content` 中确保改 framework 是 `DO Not `。在 `build phases` 中处于 `Link Binary With Libraies`

### 资源文件的获取
```objc
+ (NSData *)getImage {
    NSBundle *bundle = [NSBundle bundleForClass:[self class]];
    NSString * path = [bundle pathForResource:@"Resources" ofType:@"bundle"];
    NSData *data = [NSData dataWithContentsOfFile:[NSString stringWithFormat:@"%@/avatar.png", path]];
    return data;
}
```

## 创建 dynamic framework
- 创建 framework, `xcode -> File -> New -> Project -> iOS -> Framework/Static Library`, 确保 `build settings` 的 `Mach-O` Type 为 `Dynamic Framework`, 修改 `build libraries for distribution` 为 YES。
- 创建用于存放图片的 xcassets,` new file -> ios -> Asset Catalog`,将图片放入 xcassets 中。
- 在 finder 创建一个新的文件夹并放入其他资源文件，并添加 `.bundle` 文件后缀。然后添加到 framework 的 target。
- 确保 xcassets 和 bundle 都位于 `copy bundle ` 的 phase 中。
- 资源文件的添加与删除在 xcassets 和 bundle 中进行操作。
- 创建源文件，`Target Membership` 选择 `framework/library` 的 Target。并正确设置 头文件在 public、private 和 project 的位置，并正确的在 framework 的 头文件中 使用 import 


### 手动集成到项目中
选中要添加 framework 的 `group` -> `add files to project` -> `copy if needed、create groups`、 `add to targets`。在 target 的 `General` -> `Frameworks,Libraries,and Embedded Content` 中确保改 framework 是 Embed and Sign。在 build phases 中处于 Embed Frameworks 中。

### 资源文件的获取

获取 xcassets 中的图片
```objc
/// self 是 动态库中的一个类
+ (UIImage *)getImage {
    NSBundle *bundle = [NSBundle bundleForClass:[self class]];
    UIImage *image = [UIImage imageNamed:@"avatar" inBundle:bundle withConfiguration:nil];
    return UIImagePNGRepresentation(image);
}
```

获取 bundle 中的文件

```objc
+ (NSString *)getTxtFile {
    NSBundle *bundle = [NSBundle bundleForClass:[self class]];
    NSString *path = [bundle pathForResource:@"Resources" ofType:@"bundle"];
    NSData *data = [NSData dataWithContentsOfFile:[NSString stringWithFormat:@"%@/file.txt", path]];
    NSString *content = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    return content;
}
```

## 通过 vendored_framework 引用 framework

- pod lib create xxx 创建 pod 库。
- 将 framework 放到 pod 库文件目录中，与 .podsepc 同级。（如果是静态库相应的资源文件的 bundle 也需要放进来。可以不与 .podsepc 同级，但需要注意路径）
- 在 podspec 文件中 通过 `s.vendore_frameworks = 'xxxx.framework', 'yyyy.framework'` 引用 framework。
- 配置资源文件路径。
    - 动态库：无需关系资源文件，因为资源文件已经位于动态库内部。
    - 静态库：
        - 没有资源文件：无需进行配置
        - 有资源文件：需要添加 `s.static_framework = true`, 并在 `s.resource_bundles` 添加 `'original_static_framework_resource_bundle_name' => 'original_static_framework_resource_bundle_name.bundle/*'`，将 引用的 资源文件重新打包，注意保留文件层级并 __去除可能存在的 Info.plist__ 。否则将会保存 `multiple commamds produce Info.plist`
    - __pod 通过 vendored_framework 引用带资源的静态库，那么这个 pod 自身也必须指定为静态库，`s.static_framework = true`, 否则会造成资源文件位置不正确而导致的无法访问__
- 添加 pod 库自己的源文件和资源文件，并在 `source_files`、`public_header_files`、`resource_bundles` 正确设置。
- 如果 pod 库没有自己的源文件，则不会生成对应的 module，使用者只能使用被引用的 framework 中所定义的 module。
- 如果 pod 库有自己的源文件，则会生成 module，使用者可以 import 使用这两个。
