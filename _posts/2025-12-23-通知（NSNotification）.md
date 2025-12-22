---
layout: post
title: 通知（NSNotification）
date: 2025-12-23
tag: iOS
---

### 通知（NSNotification）
<img src="/images/posts/通知（NSNotification）/通知（NSNotification）1.jpg" > 
二、概念

NSNotification 是 iOS 中 一种广播机制，用于在不同对象之间传递消息。它属于 观察者模式（Observer Pattern） 的实现。

```
组成部分：
	1.	NSNotification：表示一条具体的通知，封装了通知的名字、发送者和附带信息。
	2.	NSNotificationCenter：通知中心，负责 注册、发送、分发通知。
	3.	Observer（观察者）：注册到通知中心的对象，收到通知后执行回调。
	4.	Notification Name：通知名称，用来区分不同通知。
```

二、代理的组成

(1) 发送通知

```
// 创建通知名字
NSString * const MyNotificationName = @"MyNotification";

// 发送通知
[[NSNotificationCenter defaultCenter] postNotificationName:MyNotificationName
                                                    object:self
                                                  userInfo:@{@"key": @"value"}];
 ```
 	•	object：发送者对象，接收者可以选择性监听某个对象的通知。
	•	userInfo：附带信息，字典形式，可以传递数据。

(2) 接收通知
方法 A：使用 Selector 回调
 ```
// 注册通知
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(handleNotification:)
                                             name:MyNotificationName
                                           object:nil];

// 回调方法
- (void)handleNotification:(NSNotification *)notification {
    NSLog(@"收到通知: %@", notification.userInfo[@"key"]);
}
 ```

 	•	name：要监听的通知名称，传 nil 表示监听所有通知。
	•	object：只监听特定对象发送的通知，传 nil 表示监听所有对象。
	•	注册通知的代码不能写在-(void)viewWillAppear:(BOOL)animated 方法中。不然 回调方法 会执行多次。

方法 B：使用 Block 回调（iOS 4 以后可用）

 ```
id observer = [[NSNotificationCenter defaultCenter] addObserverForName:MyNotificationName
                                                                object:nil
                                                                 queue:[NSOperationQueue mainQueue]
                                                            usingBlock:^(NSNotification * _Nonnull note) {
    NSLog(@"收到通知 (block)：%@", note.userInfo[@"key"]);
}];
 ```
	•	这种方式无需实现 selector，更灵活。
	•	注意：使用 block 时需要保存 observer，否则无法移除通知。
(3) 移除通知
```
// 移除指定通知
[[NSNotificationCenter defaultCenter] removeObserver:self
                                                name:MyNotificationName
                                              object:nil];

// 移除所有通知
[[NSNotificationCenter defaultCenter] removeObserver:self];
```

三、注意点
<img src="/images/posts/通知（NSNotification）/通知（NSNotification）2.jpg" > 

	1.	线程问题：通知发送和接收不保证在同一线程，必要时在回调里切换到主线程。
	2.	性能问题：不要频繁发送大量通知，否则会有性能消耗。
	3.	替代方案：
	•	KVO：监听属性变化
	•	Delegate / Block：一对一通信
	•	Combine / RxSwift：响应式编程

四、总结
	•	NSNotification：数据封装
	•	NSNotificationCenter：消息总线
	•	Observer：接收者
	•	postNotificationName：发送通知
	•	addObserver/removeObserver：注册/移除通知

通知机制是 松耦合通信 的利器，适合多对象广播消息场景，例如：
	•	App 内广播登录状态变化
	•	模块间共享数据更新
	•	控制器间消息传递
<img src="/images/posts/通知（NSNotification）/通知（NSNotification）.jpg" > 

<br>
转载请注明：[iTwinkle的博客](https://itwinkle.github.io/) » [通知（NSNotification）](http://iWolf.com/2025/12/通知（NSNotification）)  
   