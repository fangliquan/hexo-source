---
title: swift3.0_类和对象
tags:
  - swift3.0
categories: swift3.0
date: 2017-03-19 19:15:13
---
# 类和对象

### 和类名来创建一个类。类中属性的声明和常量、变量声明一样，唯一的区别就是它们的上下文是类。同样，方法和函数声明也一样。
<!-- more -->
```swift3
class Shape{
    var numberOfSize = 0
    func simpleDescription() -> String {
      return"A shape with \(numberOfSize) sides."
    }

    func simpleDescription(str:String) -> String {
        return"A shape with \(numberOfSize) sides."
    }

}
```

### 要创建一个类的实例，在类名后面加上括号。使用点语法来访问实例的属性和方法。
```swift3
var shape = Shape()
shape.numberOfSize = 7
var shapeDescription = shape.simpleDescription()
class NameShape{
    var numberOfSides:Int = 0
    var name:String
    init(name:String) {
        self.name = name;
    }
    func simpleDescription() -> String {
        return"A shape with \(numberOfSides)"
    }
}
```

### 注意self 被用来区别实例变量。当你创建实例的时候，像传入函数参数一样给类传入构造器的参数。每个属性都需要赋值——无论是通过声明（就像numberOfSides ）还是通过构造器（就像name ）。 如果你需要在删除对象之前进行一些清理工作，使用deinit 创建一个析构函数。子类的定义方法是在它们的类名后面加上父类的名字，用冒号分割。创建类的时候并不需要一个标准的根类，所以你可以忽略父类。
### 子类如果要重写父类的方法的话，需要用override 标记——如果没有添加override 就重写父类方法的话编译器会报错。编译器同样会检测override 标记的方法是否确实在父类中
```swift3
class Square:NameShape{

    var sideLength:Double = 0.0;

    init(sideLength:Double,name:String) {

        super.init(name: name)
        self.sideLength = sideLength
        numberOfSides = 4
     }
    func area() -> Double {
        return sideLength*sideLength
    }
    override func simpleDescription() -> String {
        return "A square with sides of length \(sideLength)"
    }
}
let test = Square(sideLength: 5.2, name: "my test square")
test.area()
test.simpleDescription()
```
### 除了储存简单的属性之外，属性可以有 getter 和 setter 。
```swift3
class EquilateralTriangle:NameShape{
    var sideLength: Double = 0.0
    init(sideLength:Double,name:String) {
        super.init(name: name)
        self.sideLength = sideLength;
        numberOfSides = 3
    }
    var perimeter :Double{
        get{
            return 3.0 * sideLength
         }
        set{
            sideLength = newValue/3.0
        }
    }
    override func simpleDescription() -> String {
        return "an equilateral triagle with sides of length \(sideLength)."
    }
}
var triangle  = EquilateralTriangle(sideLength: 3.1, name: "a triangle")
print(triangle.perimeter)
triangle.perimeter = 9.9
print(triangle.sideLength)
```

### 在perimeter 的 setter 中，新值的名字是newValue 。你可以在set 之后显式的设置一个名字。注意EquilateralTriangle 类的构造器执行了三步：
* 1. 设置子类声明的属性值
* 2. 调用父类的构造器
* 3. 改变父类定义的属性值。其他的工作比如调用方法、getters 和 setters 也可以在这个阶段完成。
如果你不需要计算属性，但是仍然需要在设置一个新值之前或者之后运行代码，使用willSet 和didSet 。
```swift3
class TriangleAndSquare {
    var triangle: EquilateralTriangle {
       willSet {
           square.sideLength = newValue.sideLength
        }
    }
    var square:Square{
        willSet{
            triangle.sideLength = newValue.sideLength
        }
    }

    init(size:Double,name:String) {
        square = Square(sideLength: size, name: name)
        triangle = EquilateralTriangle(sideLength: size, name: name)
    }

}
var triangleAndSquare = TriangleAndSquare(size: 10, name: "another test shape")
print(triangleAndSquare.square.sideLength)
print(triangleAndSquare.triangle.sideLength)
triangleAndSquare.square = Square(sideLength: 50, name: "larger square")
print(triangleAndSquare.triangle.sideLength)

```
### 处理变量的可选值时，你可以在操作（比如方法、属性和子脚本）之前加? 。如果? 之前的值是nil ， ? 后面的东西都会被忽略，并且整个表达式返回nil 。否则， ? 之后的东西都会被运行。在这两种情况下，整个表达式的值也是一个可选值。
```swift3
let optionalSquare:Square? = Square(sideLength: 2.5, name: "optional Square")

let sideLength = optionalSquare?.sideLength
```
