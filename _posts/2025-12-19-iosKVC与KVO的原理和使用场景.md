---
layout: post
title: KVC与KVO的原理和使用场景
date: 2025-12-19
tag: iOS
---

### KVC（Key-Value Coding）底层原理

一、KVC 是什么（官方一句话）。通过字符串 key，间接访问对象属性的一种机制。

```
[obj setValue:@18 forKey:@"age"];
NSNumber *age = [obj valueForKey:@"age"];
```


二、 KVC 底层查找顺序（⭐️必背）

1、setValue:forKey: 查找流程

假设 key = age

```
1. setAge:
2. _setAge:
3. setIsAge:
4. 直接访问实例变量（若 allowsDirectAccess == YES）
   - _age
   - _isAge
   - age
   - isAge
5. 找不到 → setValue:forUndefinedKey:

```

2、valueForKey: 查找流程

```
1. getAge
2. age
3. isAge
4. _age
5. _isAge
6. age
7. isAge
8. 找不到 → valueForUndefinedKey:

```

3、KVC 直接修改私有变量（为什么能绕过 setter）

```
@interface Person : NSObject {
    int _age;
}
@end

[p setValue:@18 forKey:@"age"];
```
✔️ 原因：

	•	KVC 通过 runtime 直接访问 ivar
	•	本质是 object_setIvar

4、KVC 本质原理（底层）

核心：Objective-C Runtime
	•	class_copyMethodList
	•	class_copyIvarList
	•	object_setIvar
	•	objc_msgSend

👉 KVC = runtime + 字符串反射

5、KVC 常见异常

```
1. key 不存在
2. 未实现 setValue:forUndefinedKey:

```

### KVO（Key-Value Observing）底层原理（⭐️重点）
1、KVO 是什么
一种 基于 KVC 的观察者模式实现

```
[person addObserver:self
         forKeyPath:@"age"
            options:NSKeyValueObservingOptionNew
            context:nil];
```

2、KVO 的本质原理（一句话）
Runtime 动态生成子类 + isa 指针指向 + 重写 setter

3、KVO 触发时系统做了什么（完整流程）

```
Person *p = [Person new];
[p addObserver:self forKeyPath:@"age" ...];
```

4、底层步骤

```
1. 动态创建子类
   NSKVONotifying_Person

2. 修改对象 isa 指针
   Person → NSKVONotifying_Person

3. 重写 setAge:
   - willChangeValueForKey:
   - 调用 super setAge:
   - didChangeValueForKey:

4. 观察者收到回调
```
 isa 指向的是子类，而不是 Person

 5、KVO 动态子类结构（面试画图）

```
 Person
   ↑
NSKVONotifying_Person
   - 重写 setter
   - 重写 class 方法
   - 重写 dealloc
```

为什么重写 class？

```
NSLog(@"%@", object_getClass(p));      // NSKVONotifying_Person
NSLog(@"%@", [p class]);               // Person
```

6、KVO 为什么一定要走 setter？

```
_age = 10;   // ❌ 不触发
self.age = 10; // ✅ 触发
```

✔️ 因为：
	•	KVO 是通过 重写 setter 实现的
	•	没有 setter → 不生效
7、手动触发 KVO（高级）

```
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
    return NO;
}
```

```
[self willChangeValueForKey:@"age"];
_age = 10;
[self didChangeValueForKey:@"age"];

```

### 标准面试“背诵版总结”（⭐️）

```
KVC 是通过字符串 key，基于 runtime 查找方法和 ivar，实现属性的间接访问。
KVO 是在 KVC 基础上，利用 runtime 动态生成子类、修改 isa 指针并重写 setter，在属性变化前后插入通知逻辑，从而实现观察者模式。
```

### KVC 在项目中的实际使用场景

场景 1️⃣ 字典 → Model 映射（最常见）

接口返回 JSON：

```
{
  "id": 1001,
  "name": "Twinkle",
  "age": 18
}
```

使用方式

```
@implementation Person
- (void)setValue:(id)value forUndefinedKey:(NSString *)key {}
@end

Person *p = [Person new];
[p setValuesForKeysWithDictionary:json];
```

场景 2️⃣ 处理接口字段与属性名不一致
{
  "id": 1001
}

```
@interface Person : NSObject
@property (nonatomic, assign) NSInteger personId;
@end

```

KVC 转换

```
- (void)setValue:(id)value forKey:(NSString *)key {
    if ([key isEqualToString:@"id"]) {
        self.personId = [value integerValue];
        return;
    }
    [super setValue:value forKey:key];
}
```

场景 3️⃣ 访问 / 修改私有属性（框架内部）

```
[p setValue:@(YES) forKey:@"_isPlaying"];

```

### KVO 在项目中的真实使用场景（⭐️重点）

场景 1️⃣ ViewModel → View 数据绑定（MVVM）

背景

```
@property (nonatomic, copy) NSString *title;
```

View 监听 ViewModel
```
[self.viewModel addObserver:self
                 forKeyPath:@"title"
                    options:NSKeyValueObservingOptionNew
                    context:nil];

```

```
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context {
    self.titleLabel.text = change[NSKeyValueChangeNewKey];
}

```
在 MVVM 架构中，KVO 用于 ViewModel 驱动 View 更新




<br>
转载请注明：[iWolf的博客](https://itwinkle.github.io/) » [KVC与KVO的原理和使用场景](http://iWolf.com/2025/12/KVC与KVO的原理和使用场景/)  
