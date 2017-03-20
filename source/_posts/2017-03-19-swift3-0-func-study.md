---
title: swift3.0_函数和闭包
tags:
  - swift3.0
categories: swift3.0
date: 2017-03-19 17:15:13
---

# 函数和闭包
### 使用func 来声明一个函数，使用名字和参数来调用函数。使用-> 来指定函数返回值的类型

``` swift3
func great( person:String,day :String) ->String{
    return "Hello \(person),today is \(day)"
}
```
<!-- more -->

### 默认情况下，函数使用它们的参数名称作为它们参数的标签，在参数名称前可以自定义参数标签，或者使用_表示不使用参数标签。
``` swift3
func greaton(_ person:String,on day :String) ->String{
    return "Hello \(person),today is \(day)"
}
great(person: "", day: "")

greaton("John", on: "Sunday")
```
### 使用元组来让一个函数返回多个值。该元组的元素可以用名称或数字来表示。
``` swift3
func calculateStatistics(scores:[Int]) ->(min:Int,max:Int,sum:Int){
    var minInt = scores[0]
    var maxInt = scores[0]
    var sum = 0

    for score in scores {
        if score>maxInt {
            maxInt = score
        }else if score<minInt{
            minInt = score
        }
        sum+=score
    }
    return(minInt,maxInt,sum)
}

let statistics = calculateStatistics(scores: [3,4,567,89,99])

print(statistics.sum)

print(statistics.max)

print(statistics.min)

print(statistics.2)
```
###  函数可以带有可变个数的参数，这些参数在函数内表现为数组的形式：
``` swift3
func sumOf(numbers:Int...)->Int{
    var sum = 0;
    for number in numbers {
        sum+=number
    }
    return sum
}

sumOf(numbers: 34,56,78)

sumOf()
```
###  函数可以嵌套。被嵌套的函数可以访问外侧函数的变量，你可以使用嵌套函数来重构一个太长或者太复杂的函数。
``` swift3
func returnFifteen()->Int{
    var y = 10
    func add(){
        y+=10
    }
    add();
    return y;
}
returnFifteen()
```
###  函数是第一等类型，这意味着函数可以作为另一个函数的返回值。
``` swift3
func makeIncrementer()->((Int) ->Int){
    func addOne(number:Int)->Int{
        return 1+number
    }
    return addOne
}

var increment = makeIncrementer()

increment(10)
```

###  函数也可以当做参数传入另一个函数
``` swift3
func hasAnyMatches(list:[Int],condition:(Int) ->Bool)->Bool{
    for item in list {
        if condition(item) {
            return true;
        }
    }
    return false;
}

func lessThanTen(number:Int)->Bool{
    return number<10
}

var numbers = [20,19,6,12]

hasAnyMatches(list: numbers, condition: lessThanTen)
```

###  函数实际上是一种特殊的闭包:它是一段能之后被调取的代码。闭包中的代码能访问闭包所建作用域中能得到的变量和函数，即使闭包是在一个不同的作用域被执行的 - 你已经在嵌套函数例子中所看到。你可以使用{} 来创建一个匿名闭包。使用in 将参数和返回值类型声明与闭包函数体进行分离。
``` swift3

numbers.map { (number :Int) -> Int in
    let result = 3*number;
    return result;
}

print(numbers)
```

###  有很多种创建更简洁的闭包的方法。如果一个闭包的类型已知，比如作为一个回调函数，你可以忽略参数的类型和返回值。单个语句闭包会把它语句的值当做结果返回。

``` swift3
let mappedNumbers = numbers.map({number in 3*number})

print(mappedNumbers)

```

###  你可以通过参数位置而不是参数名字来引用参数——这个方法在非常短的闭包中非常有用。当一个闭包作为最后一个参数传给一个函数的时候，它可以直接跟在括号后面。当一个闭包是传给函数的唯一参数，你可以完全忽略括号。
``` swift3
let sortedNumbers = numbers.sort{$0>$1}
print(sortedNumbers)

```
