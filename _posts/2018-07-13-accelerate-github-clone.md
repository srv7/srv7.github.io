---
layout: post
title: 给 github clone 加速
date: 2018-07-13 15:43:08
categories: 其他
---

![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft8adwk3sfj20jq0d7k44.jpg)

<!-- more -->


虽然 github 在大陆没有被墙，但访问的速度依然很慢。在 clone 稍大一点的项目时就要了命了，Google 之后有人说把 `postBuffer(智能HTTP传输所使用的缓冲区)` 加大一些，于是就更改了下配置：

```
git config --global http.postBuffer 524288000
```

单位为 `Byte`, `524288000B` 也就是 `500M`, 添加了之后速度也没有明细的提升。

如果说是科学上网的话能不能解决问题呢？打开 `ss`、 打开全局代理，速度依然没上去。

Google 后发现 git 默认是不会使用你的代理的，需要配置让 git 去使用你的代理。

命令如下：

```
// 别着急操作，下面有更好的方式
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```

其中 `1080` 为 HTTP 代理监听端口。改为你自己的端口号。每个人可能不一样，比如我的就是`1087`
![HTTP](https://ws1.sinaimg.cn/large/b92f96b9gy1ft894qbzibj20dc0b8gm6.jpg)

除了 `HTTP` 还可以使用 `Socks5` 协议, 效果是一样的。

```
git config --global http.proxy socks5://127.0.0.1:1086
git config --global https.proxy socks5://127.0.0.1:1086
```
> 这里的端口是 `socks5` 监听端口
![Socks5](https://ws1.sinaimg.cn/large/b92f96b9gy1ft89nxjymtj20dc0b83zc.jpg)

**但同时，也请注意，这里指的是https协议，也就是**
```
git clone https://www.github.com/xxxx/xxxx.git
```
**这种**。

**对于SSH协议，也就是**
```
git clone git@github.com:xxxxxx/xxxxxx.git
```
**这种，依旧是无效的**。

---

上面的方法是配置 git 对所有的仓库都使用代理，这样的话如果需要克隆coding之类的国内仓库，会奇慢无比，所以不推荐使用上面的方法设置。

只有在 clone 位于 github 的仓库时才使用代理，其他时候不使用代理，对国内的 coding、gitee以及自建的 gitlab 也就没有影响了。

命令如下：

```
// 1080 改为自己的 HTTP 监听端口
git config --global http.https://github.com.proxy https://127.0.0.1:1080
git config --global https.https://github.com.proxy https://127.0.0.1:1080

```
或

```
//  1086 改为自己的 socks5 监听端口
git config --global http.https://github.com.proxy socks5://127.0.0.1:1086
git config --global https.https://github.com.proxy socks5://127.0.0.1:1086
```

clone `swift` 仓库的速度
![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft8adwk3sfj20jq0d7k44.jpg)

clone `linux` 内核仓库的速度对比
![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft8ag2rrl4j20jq0d7tl8.jpg)

下面两行命令可以取消代理：

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```


