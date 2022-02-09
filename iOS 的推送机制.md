# iOS 的推送机制

## 1. 概述

对于 iOS / iPadOS 开发来说，一个 app 拥有必须的通知机制是必不可少的。通知可以在用户将应用切到前台或者后台的时候还可以提示必要的信息。作为开发者，必须充分利用系统的通知功能。

![](https://pic.imgdb.cn/item/6203207f2ab3f51d91bd5206.jpg)

## 2. 通知的分类

iOS / iPadOS 主要使用 `User Notification` 框架进行推送通知。通知主要分为 **本地通知** 和 **远程通知**。其中，本地通知是由 app 生成通知内容并且指定发送条件，比如时间或地址，去作为触发推送的条件；远程通知是由公司的 server 生成通知内容，指定发送条件，然后由 `Apple Push Notification service (APNs)` 统一分发给用户的设备。

本地通知和远程通知都有广泛的应用，主要区别是展示的条件和应用场景不同。本地通知只有在 app 在后台运行或者在 app 被关闭时才可以展示。否则，通知仍然被成功加入到通知中心中，但是并不会弹出横幅、提示声音。远程通知则可以在 app 任何运行状态下显示出来。

本地通知推荐本地工具类推送一些不会在马上生效的通知时使用。而需要马上生效的通知，建议通过提示框展示。而远程通知更适合于新闻、聊天类应用，需要在任何时刻可以强通知用户的场景。

这篇文章先简要介绍一下本地通知如何发送。

## 3. 发送本地通知

发送本地通知相比发送远程通知简单很多，由于 Apple 良好的接口设计，接口可读性很强，基本上看接口就知道这个方法要做什么事情。

Apple 的开发者文档也很不错，向讲故事一样，只要给定一个主题从头到尾看一遍就好。

### 3.1 向系统申请通知权限

要发送本地通知，必须在发送通知前向系统为 app 申请通知权限。申请权限的位置倒是无所谓，同意一次以后就不需要再次申请，之后通知权限由系统管理。我们需要用到 `UNUserNotificationCenter` 里的 `requestAuthorization` 方法。

```swift
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound], completionHandler: {
            (success, error) in
            if success {
                print("success")
            }
            if error != nil {
                print(error!.localizedDescription)
            }
        })
```

### 3.2 构造通知请求

#### 3.2.1 构造通知内容

对于可以改变标题、显示内容的通知，我们必须使用 `UNMutableNotificationContent` 类来构造通知内容。

```swift
let notificationContent = UNMutableNotificationContent()
notificationContent.title = "yourTitle"
notificationContent.sound = UNNotificationSound.default
```

还有一个相似的 `UNNotificationContent` 类，不过它的通知是不可以编辑标题、声音的。

#### 3.2.2 构造触发器

触发器对于发送通知也是必须的，触发器确定了发送的条件。iOS / iPadOS 支持很多种触发器，这里由一个简单的时间触发器 `` 举例。

```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 10, repeats: false)
```

timeInterval 参数规定的倒计时间是以秒为单位的，repeat 表示发出通知以后是否还会重复发送，对于不需要强提醒的场景来说，一般设置为 false。到点提醒一次就好。

对于我的应用，我使用了这样的语句，用来规定一个类似于提醒事项的闹钟通知。

```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: dueDate.timeIntervalSinceNow, repeats: false)
```

#### 3.2.3 构造请求

设置好内容、触发器，然后将通知赋予一个能够在通知中心唯一识别的 id，就可以为这个通知构造请求了。

```swift
let request = UNNotificationRequest(identifier: String(self.TodoList[id].id), content: notificationContent, trigger: trigger)
```

注意，这里的 identifier 一定要保证每个通知都可以有唯一标识。

### 3.3 将请求加入通知中心的消息队列

构造好请求后将请求加入通知中心的消息队列，然后，在满足 trigger 的条件后如果 app 不在前台运行就会展示一个通知。

```swift
UNUserNotificationCenter.current()
            .add(request)
```

## 4. 其他内容

有关通知的其他内容还有很多很多，对于开发来说，用多了以后其实就是那么几个，剩下的如果到用的时候直接看文档就好了。Apple 的开发者文档和其他公司的不太一样，感觉像是个文章一样，刚看起来有些费劲，但是适应了它的写作逻辑以后就明白其实是很易读的。



