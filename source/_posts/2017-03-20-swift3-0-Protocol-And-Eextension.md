---
title: swift3.0_协议(Protocal)和扩展(Extension)
tags:
  - swift3.0
categories: swift3.0
date: 2017-03-20 18:13:44
---

## 协议(Protocal
#### 使用protocol 来声明一个协议。
```swift3
 ExampleProtocal{
    var simpleDesp:String{get}
    mutating func adjust()

}
```
<!-- more -->

#### 类、枚举和结构体都可以实现协议。
```swift3
class SimpleClass: ExampleProtocal{
    var simpleDesp: String = "A simple class"
    var anotherProperty:Int = 45621
    func adjust() {
        simpleDesp += " NO 100% adjusted"
    }

}
```

##### 注意声明SimpleStructure 时候mutating 关键字用来标记一个会修改结构体的方法。SimpleClass 的声明不需要标记任何方法，因为类中的方法通常可以修改类属性（类的性质）。
## 使用extension
#### 来为现有的类型添加功能，比如新的方法和计算属性。你可以使用扩展在别处修改定义，甚至是从外部库或者框架引入的一个类型，使得这个类型遵循某个协议。
```swift3
var simple = SimpleClass()
simple.adjust()

let aDesp = simple.simpleDesp

struct SimpleStructure:ExampleProtocal{
    var simpleDesp: String = "A simple structure"
    mutating func adjust() {
        simpleDesp += "(adjusted)"
    }

}
var b = SimpleStructure()
b.adjust()
let bDesp = b.simpleDesp

extension Int:ExampleProtocal {
    var simpleDesp :String {
        return "The number\(self)"
    }
    mutating func adjust() {
        self += 42
    }

}

print(7.simpleDesp)
```

#### 你可以像使用其他命名类型一样使用协议名——例如，创建一个有不同类型但是都实现一个协议的对象􀔀合。当你处理类型是协议的值时，协议外定义的方法不可用。
```swift3
let protocolValue : ExampleProtocal = simple
print(protocolValue.simpleDesp)
//print(protocolValue.anotherProperty) // 去掉注释可以看到错误
```
#### 即使protocolValue 变量运行时的类型是simpleClass ，编译器会把它的类型当做ExampleProtocol 。这表示你不能调用类在它实现的协议之外实现的方法或者属性。
