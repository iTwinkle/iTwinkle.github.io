---
layout: post
title: 扩展（Extension）
date: 2025-12-22
tag: iOS
---

### 扩展（Extension）

一、iOS Extension 是什么？

Class Extension 是在编译期向类本身追加私有成员（属性 / 方法）的机制
核心结论（先记住）：
✅ Class Extension = 类的一部分，不是外挂

典型写法
```
@interface Person ()
@property (nonatomic, copy) NSString *name;
- (void)privateMethod;
@end
```
特征：
	•	没有名字：()
	•	一般写在 .m
	•	对外不可见
	•	参与类的编译
二、Class Extension 到底“扩展”了什么？

1️⃣ 能扩展的内容（完整列表）

```
类型           是否支持
私有方法          ✅
私有属性          ✅
只读 → 读写       ✅
协议声明          ✅
ivar（本质）      ✅
```

```
@interface PlayerManager ()
- (void)prepare;
@end
```

```
@interface PlayerManager ()
@property (nonatomic, strong) AVPlayer *player;
@end
```

```
编译器会做什么？
自动生成 ivar：_player，并直接并入类结构
```

.h
```
@interface Task : NSObject
@property (nonatomic, copy, readonly) NSString *taskId;
@end
```

.m
```
@interface Task ()
@property (nonatomic, copy, readwrite) NSString *taskId;
@end
```

三、Class Extension 的“编译期本质”（核心原理）

❗关键点：
Class Extension 在编译期就被合并进类的 @interface

编译后等价于：
```
@interface Person : NSObject {
    NSString *_name;
}
- (void)privateMethod;
@end
```

所以它具备：
	•	真正的 ivar
	•	真正的方法列表
	•	不依赖 runtime 动态合并

四、Class Extension vs Category（从底层讲清楚）

<img src="/images/posts/扩展（Extension）/扩展（Extension）.jpg" > 
⚠️ 为什么 Category 不能加 ivar？

因为：

Category 是 运行时才合并到类的方法列表
而 ivar 布局在 编译期已确定


五、Class Extension 在 runtime 层面长什么样？
我们从 runtime 结构看：

```
struct objc_class {
    Class isa;
    Class superclass;
    cache_t cache;
    class_data_bits_t bits;
}
```

Class Extension 的成员：
	•	属性 → ivar → class_rw_t
	•	方法 → method_list_t

👉 直接进 class_rw_t

Category：
	•	方法列表
	•	在 runtime 加载时 append 到 method list

六、为什么 Class Extension “必须写在 .m”？
因为它的目的就是 “不对外暴露”

如果写在 .h：
	•	所有 import 的人都能看到
	•	那就失去了“私有”的意义

📌 真正的规则是：

谁能看到头文件，谁就能看到 Extension

不是语法限制，是设计目的。

Class Extension 是 Objective-C 中唯一一种「安全、编译期、真正意义上的私有扩展机制」


<br>
转载请注明：[iTwinkle的博客](https://itwinkle.github.io/) » [扩展（Extension）](http://iWolf.com/2025/12/扩展（Extension）)  
   