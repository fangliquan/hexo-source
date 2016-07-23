---
title: iO9新特性学习--泛型
tags:
  - runtime
  - iOS9新特性
categories: iOS
date: 2016-07-23 17:06:11
---


# 泛型：限制类型
## 使用场景
1. 在集合（数组，字典，NSSet)中使用泛型比较常见
2. 当声明一个类，类里面的某些书写的类型不确定，这时候我们才使用泛型。

<!-- more -->

## 书写规范
1. 在类型后面定义泛型，
```objc
NSMutableArray<UIImage *> *mutableArray
```

## 修饰：
只能修饰方法的调用
## 好处：
1. 提高规范，减少交流
2. 通过集合取出来对象，直接当做泛型对象使用，可以直接使用点语法
```objc
self.mutableArray[0].temp;
```

## 子类想给父类赋值使用协变
`__covariant`(协变）:用于数据强转类型，可以向上强转，子类，可以转成 父类，例:

```objc
@interface Person<__covariant ObjectType>
```

## 父类强转成子类 逆变
`__contravariant`（逆变）：用于泛型数据强转类型，可以向下强转，父类可以转成子类 例:

```objc
@interface Person<__contravariant ObjectType>
```

## 定义泛型类

```objc
@interface Person<__covariant ObjectType> : NSObject
@property(nonatomic) ObjectType language;
@end
```
