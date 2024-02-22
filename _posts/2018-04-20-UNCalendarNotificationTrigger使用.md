---
layout: post
title: UNCalendarNotificationTrigger使用
subtitle: CalendarNotificationTrigger
date: 2018-04-20 13:35:36
categories: iOS
published: false
---
最近在做一款提醒类软件，一个提醒流包含多个节点，每个节点都需要在用户设定的时间来提醒用户。提醒分为四种类型：

###### 在线提醒
靠 APNS 发推送来提醒
###### 定时提醒
不依托 APNS，App 自己在指定时间点发出本地推送来提醒用户
###### 定距提醒
不依托 APNS，App 自己在指定时间点发出本地推送来提醒用户， 定距为每个节点之间的时间间隔，时间间隔由用户自行定义
###### 循环提醒
不依托 APNS，App 自己在指定时间点发出本地推送来提醒用户，又分为四种类型：按年循环、按月循环、按周循环、按每天循环，其中按年循环支持农历

为了用户收到提醒点击推送时能打开 App, 提高用户粘性，没有使用 `EventKit` 来实现，而是使用了 Apple 在 iOS 10 中推出的 `UserNotifications Framework`.

关于`UserNotifications`请看
- [玩转 iOS 10 推送 —— UserNotifications Framework（上）](https://www.jianshu.com/p/2f3202b5e758)
- [玩转 iOS 10 推送 —— UserNotifications Framework（中）](https://www.jianshu.com/p/5a4b88874f3a)
- [玩转 iOS 10 推送 —— UserNotifications Framework（下）](https://www.jianshu.com/p/25ca24215f75)

下面就简单说下如何使用 `UserNotifications Framework` 来实现这些功能。主要用到了 `UNCalendarNotificationTrigger` 类

### 在线提醒
向后端发起 `post` 请求，`body`中带上用户设置的时间就行，很简单

### 定时、定距提醒
使用 `UNCalendarNotificationTrigger`创建一个本地推送，并添加到 `NotificationCenter`。

```
    // 创建推送内容
    let content = UNMutableNotificationContent()
    content.title = "title"
    content.subtitle = "body"
    content.body = "body"
    content.sound = UNNotificationSound.default()
    content.userInfo = ["key": "value"]
    
    // 计算 date
    let date = ...// Date set by the user，定距提醒每个节点的 date 需要额外计算得出具体的 date
    
    // 创建触发器所需要的 dateComponents
    let componentSet: Set<Calendar.Component> = [.year, .month, .day, .hour, .minute, .second]
    dateComponents =  calendar.dateComponents(componentSet, from: date)
    
    // 创建推送触发器
    let trigger =  UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: false)
    
    // 创建 request
    let request = UNNotificationRequest(identifier: "identifier", content: content, trigger: trigger)
    
    // 添加到 UNUserNotificationCenter
    UNUserNotificationCenter.current().add(request, withCompletionHandler: { (error) in
        if let _ = error {
            fatalError("添加提醒出错")
        }
    })
```

### 循环提醒
代码与上面类似

```
    var calendar: Calendar
    if 农历 {
        calendar = Calendar(identifier: .chinese)
    } else {
        calendar = Calendar(identifier: .gregorian)
    }
        
    var dateComponents: DateComponents
    
    switch node.loopChoice {
    case Constant.DAY: // 按天循环，只需要设置 时分秒
        let set: Set<Calendar.Component> = [.hour, .minute]
        dateComponents = calendar.dateComponents(set, from: date)
    case Constant.WEEK: // 按周循环 需要设置 周几、时分秒，1 是周日，7 是周六
        let set: Set<Calendar.Component> = [.hour, .minute]
        dateComponents = calendar.dateComponents(set, from: date)
        dateComponents.weekday = ... // e.g. 3(每周二)
    case Constant.MONTH: // 按月循环 需要设置 日时分秒 
        let set: Set<Calendar.Component> = [.hour, .minute]
        dateComponents = calendar.dateComponents(set, from: date)
        dateComponents.day = .../ e.g. 29 (每月的29日)
    case Constant.YEAR: // 按年循环 需要设置 月日时分秒 日历的话需要设置isLeapMonth = true
        let set: Set<Calendar.Component> = [.hour, .minute]
        dateComponents = calendar.dateComponents(set, from: date)
        dateComponents.month = ... // 4 (每年 4 月)
        dateComponents.day = ... // e.g. 29 (每月的29日)
        dateComponents.isLeapMonth = .. // 是否是农历
    default:
        dateComponents = calendar.dateComponents([.year], from: date)
    }
```

如果说设置的每年2月29日提醒，当年2月份没有29日，则会在3月1日提醒，规则如下

![](https://ws1.sinaimg.cn/large/b92f96b9gy1fqj3vjcgrzj20jg0fpgnx.jpg)
