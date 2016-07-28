---
title: iOS开发--多线程 NSThread GCD 学习
tags:
  - iOS多线程
  - NSThread
  - GCD
date: 2016-07-28 23:26:17
categories: iOS
---
### 1.基本概念
- 1.1 进程

    进程是指在系统中正在运行的一个应用程序。每个进程之间是独立的，每个进程均运行在其专用且受保护的内存空间内。
<!--more -->

- 1.2 线程

（1）基本概念

    1个进程要想执行任务，必须得有线程（每1个进程至少要有1条线程），线程是进程的基本执行单元，一个进程（程序）的所有任务都在线程中执行。
（2）线程的串行

     1个线程中任务的执行是串行的，如果要在1个线程中执行多个任务，那么只能一个一个地按顺序执行这些任务。也就是说，在同一时间内，1个线程只能执行1个任务。

- 1.3 多线程

（1）基本概念

    即1个进程中可以开启多条线程，每条线程可以并行（同时）执行不同的任务。
（2）线程的并行

    并行即同时执行。比如同时开启3条线程分别下载3个文件（分别是文件A、文件B、文件C。
（3）多线程并发执行的原理

     在同一时间里，CPU只能处理1条线程，只有1条线程在工作（执行）。多线程并发（同时）执行，其实是CPU快速地在多条线程之间调度（切换），如果CPU调度线程的时间足够快，就造成了多线程并发执行的假象
（4）多线程优缺点

    优点
        1）能适当提高程序的执行效率。
        2）能适当提高资源利用率（CPU、内存利用率）

    缺点
        1）开启线程需要占用一定的内存空间（默认情况下，主线程占用1M，子线程占用512KB），如果开启大量的线程，会占用大量的内存空间，降低程序的性能。
        2）线程越多，CPU在调度线程上的开销就越大。
        3）程序设计更加复杂：比如线程之间的通信、多线程的数据共享

- 1.4 多线程在iOS开发中的应用

（1）主线程

        1）一个iOS程序运行后，默认会开启1条线程，称为“主线程”或“UI线程”。
        2）作用。刷新显示UI,处理UI事件。
（2）使用注意

        1）不要将耗时操作放到主线程中去处理，会卡住线程。
- 1.5 iOS中多线程的实现方案

（1）`pthread`

	01 特点：
	（1）一套通用的多线程API
	（2）适用于Unix\Linux\Windows等系统
	（3）跨平台\可移植
	（4）使用难度大

	02 使用语言：c语言
	03 使用频率：几乎不用
	04 线程生命周期：由程序员进行管理

（2） `NSThread`

	01 特点：
	（1）使用更加面向对象
	（2）简单易用，可直接操作线程对象

	02 使用语言：OC语言
	03 使用频率：偶尔使用
	04 线程生命周期：由程序员进行管理

（3）`GCD`

	01 特点：
	（1）旨在替代NSThread等线程技术
	（2）充分利用设备的多核(自动)

	02 使用语言：OC语言
	03 使用频率：经常使用
	04 线程生命周期：自动管理

(4) `NSOperation`

	01 特点：
	（1）基于GCD（底层是GCD）
	（2）比GCD多了一些更简单实用的功能
	（3）使用更加面向对象

	02 使用语言：OC语言
	03 使用频率：经常使用
	04 线程生命周期：自动管理


---

### 2.pthread

（1）pthread的基本使用（需要包含头文件）
```objc
    //使用pthread创建线程
   pthread_t thread;
    NSString *name = @"wendingding";
    //使用pthread创建线程
    //第一个参数：线程对象地址
    //第二个参数：线程属性
    //第三个参数：指向函数的执行
    //第四个参数：传递给该函数的参数
    pthread_create(&thread, NULL, run, (__bridge void *)(name));
```

---

### 3.NSThread
（1）NSThread的基本使用
```objc
//第一种创建线程的方式：alloc init.
//特点：需要手动开启线程，可以拿到线程对象进行详细设置
    //创建线程
    /*
     第一个参数：目标对象
     第二个参数：选择器，线程启动要调用哪个方法
     第三个参数：前面方法要接收的参数（最多只能接收一个参数，没有则传nil）
     */
    NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(run:) object:@"wendingding"];
     //启动线程
    [thread start];

//第二种创建线程的方式：分离出一条子线程
//特点：自动启动线程，无法对线程进行更详细的设置
    /*
     第一个参数：线程启动调用的方法
     第二个参数：目标对象
     第三个参数：传递给调用方法的参数
     */
    [NSThread detachNewThreadSelector:@selector(run:) toTarget:self withObject:@"我是分离出来的子线程"];

//第三种创建线程的方式：后台线程
//特点：自动启动县城，无法进行更详细设置
[self performSelectorInBackground:@selector(run:) withObject:@"我是后台线程"];
```
（2）设置线程的属性
```objc
   //设置线程的属性
    //设置线程的名称
    thread.name = @"线程A";

    //设置线程的优先级,注意线程优先级的取值范围为0.0~1.0之间，1.0表示线程的优先级最高,如果不设置该值，那么理想状态下默认为0.5
    thread.threadPriority = 1.0;
```
（3）线程的状态（了解）
```objc
//线程的各种状态：新建-就绪-运行-阻塞-死亡
//常用的控制线程状态的方法
[NSThread exit];//退出当前线程
[NSThread sleepForTimeInterval:2.0];//阻塞线程
[NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:2.0]];//阻塞线程
//注意：线程死了不能复生
```
（4）线程安全

        01 前提：多个线程访问同一块资源会发生数据安全问题
        02 解决方案：加互斥锁
        03 相关代码：@synchronized(self){}
        04 专业术语-线程同步
        05 原子和非原子属性（是否对setter方法加锁）

（5）线程间通信
```objc
-(void)touchesBegan:(nonnull NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event
{
//    [self download2];

    //开启一条子线程来下载图片
    [NSThread detachNewThreadSelector:@selector(downloadImage) toTarget:self withObject:nil];
}

-(void)downloadImage
{
    //1.确定要下载网络图片的url地址，一个url唯一对应着网络上的一个资源
    NSURL *url = [NSURL URLWithString:@"http://p6.qhimg.com/t01d2954e2799c461ab.jpg"];

    //2.根据url地址下载图片数据到本地（二进制数据
    NSData *data = [NSData dataWithContentsOfURL:url];

    //3.把下载到本地的二进制数据转换成图片
    UIImage *image = [UIImage imageWithData:data];

    //4.回到主线程刷新UI
    //4.1 第一种方式
//    [self performSelectorOnMainThread:@selector(showImage:) withObject:image waitUntilDone:YES];

    //4.2 第二种方式
//    [self.imageView performSelectorOnMainThread:@selector(setImage:) withObject:image waitUntilDone:YES];

    //4.3 第三种方式
    [self.imageView performSelector:@selector(setImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:YES];
}
```
（6）如何计算代码段的执行时间
```objc
//第一种方法
    NSDate *start = [NSDate date];
    //2.根据url地址下载图片数据到本地（二进制数据）
    NSData *data = [NSData dataWithContentsOfURL:url];

    NSDate *end = [NSDate date];
    NSLog(@"第二步操作花费的时间为%f",[end timeIntervalSinceDate:start]);

//第二种方法
    CFTimeInterval start = CFAbsoluteTimeGetCurrent();
    NSData *data = [NSData dataWithContentsOfURL:url];

    CFTimeInterval end = CFAbsoluteTimeGetCurrent();
    NSLog(@"第二步操作花费的时间为%f",end - start);
```
###4.GCD

（1）GCD基本知识

    01 两个核心概念-队列和任务
    02 同步函数和异步函数

（2）GCD基本使用【重点】

    01 异步函数+并发队列：开启多条线程，并发执行任务
    02 异步函数+串行队列：开启一条线程，串行执行任务
    03 同步函数+并发队列：不开线程，串行执行任务
    04 同步函数+串行队列：不开线程，串行执行任务
    05 异步函数+主队列：不开线程，在主线程中串行执行任务
    06 同步函数+主队列：不开线程，串行执行任务（注意死锁发生）
    07 注意同步函数和异步函数在执行顺序上面的差异

（3）GCD线程间通信

```objc
 //0.获取一个全局的队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

    //1.先开启一个线程，把下载图片的操作放在子线程中处理
    dispatch_async(queue, ^{

       //2.下载图片
        NSURL *url = [NSURL URLWithString:@"http://h.hiphotos.baidu.com/zhidao/pic/item/6a63f6246b600c3320b14bb3184c510fd8f9a185.jpg"];
        NSData *data = [NSData dataWithContentsOfURL:url];
        UIImage *image = [UIImage imageWithData:data];

        NSLog(@"下载操作所在的线程--%@",[NSThread currentThread]);

        //3.回到主线程刷新UI
        dispatch_async(dispatch_get_main_queue(), ^{
           self.imageView.image = image;
           //打印查看当前线程
            NSLog(@"刷新UI---%@",[NSThread currentThread]);
        });

    });
```
（4）GCD其它常用函数

```objc

    01 栅栏函数（控制任务的执行顺序）
    dispatch_barrier_async(queue, ^{
        NSLog(@"--dispatch_barrier_async-");
    });

    02 延迟执行（延迟·控制在哪个线程执行）
      dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"---%@",[NSThread currentThread]);
    });

    03 一次性代码（注意不能放到懒加载）
    -(void)once
    {
        //整个程序运行过程中只会执行一次
        //onceToken用来记录该部分的代码是否被执行过
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{

            NSLog(@"-----");
        });
    }

    04 快速迭代（开多个线程并发完成迭代操作）
       dispatch_apply(subpaths.count, queue, ^(size_t index) {
    });

    05 队列组（同栅栏函数）
    //创建队列组
    dispatch_group_t group = dispatch_group_create();
    //队列组中的任务执行完毕之后，执行该函数
    dispatch_group_notify(dispatch_group_t group,
	dispatch_queue_t queue,
	dispatch_block_t block);
```

- 学习
  [小马哥·文顶顶](http://520it.com)）
