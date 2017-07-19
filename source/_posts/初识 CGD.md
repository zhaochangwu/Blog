---
title: 初识 GCD
date: 2017-07-19 15:32:33
categories:
- iOS
tags:
- iOS
- GCD
---
# 初识 GCD
## 什么是GCD
Grand Central Dispatch或者GCD，是一套低层API，提供了一种新的方法来进行并发程序编写。从基本功能上讲，GCD有点像NSOperationQueue，他们都允许程序将任务切分为多个单一任务然后提交至工作队列来并发地或者串行地执行。GCD比之NSOpertionQueue更底层更高效，并且它不是Cocoa框架的一部分。

除了代码的平行执行能力，GCD还提供高度集成的事件控制系统。可以设置句柄来响应文件描述符、mach ports（Mach port 用于 OS X上的进程间通讯）、进程、计时器、信号、用户生成事件。这些句柄通过GCD来并发执行。

GCD的API很大程度上基于block，当然，GCD也可以脱离block来使用，比如使用传统c机制提供函数指针和上下文指针。实践证明，当配合block使用时，GCD非常简单易用且能发挥其最大能力。
你可以在Mac上敲命令“man dispatch”来获取GCD的文档。

## Dispatch Objects
尽管GCD是纯c语言的，但它被组建成面向对象的风格，GCD对象被称为dispatch object。Dispatch object像Cocoa对象一样是引用计数的。使用dispatch_release和dispatch_retain函数来操作dispatch object的引用计数来进行内存管理

## Dispatch Queues
GCD的基本概念就是dispatch queue。dispatch queue是一个对象，它可以接受任务，并将任务以先到先执行的顺序来执行。dispatch queue可以是并发的或串行的。并发任务会像NSOperationQueue那样基于系统负载来合适地并发进行，串行队列同一时间只执行单一任务。

### GCD中有三种队列类型：
#### 1 The main queue:
与主线程功能相同。实际上，提交至main queue的任务会在主线程中执行。main queue可以调用dispatch_get_main_queue()来获得。因为main queue是与主线程相关的，所以这是一个串行队列。
```objective-c
dispatch_queue_t mainQueue = dispatch_get_main_queue();
```
#### 2 Global queues:
全局队列是并发队列，并由整个进程共享。进程中存在三个全局队列：高、中（默认）、低三个优先级队列。可以调用dispatch_get_global_queue函数传入优先级来访问队列。

```objective-c
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW,0);
```
#### 3 用户创建的队列:
没有一个特定的名字来形容这种队列，这些的队列是用函数 dispatch_queue_create 创建的队列.
```objective-c
dispatch_queue_t queue1 = dispatch_queue_create("com.apple.queue", DISPATCH_QUEUE_CONCURRENT);
```
>第一个参数是标签，Apple建议我们使用倒置域名来命名队列，比如“com.dreamingwish.subsystem.task”。这些名字会在崩溃日志中被显示出来，也可以被调试器调用，这在调试中会很有用。

* Concurrent：
1 用来执行大量的并行任务，GCD自动创建了四种全局的Concurrent队列，他们只是优先级不同：
2 因为这队列是全局的，不需要retain和release，掉用也会被忽略
3 还可以创建自己的并行队列
* Serial：
1 串行队列会确保每个任务按顺序执行
2 尽量使用并行队列，而不是去创建多个串行队列

### dispatch queue API
#### dispatch_async（非同步）
block提交到queue之后会继续执行队列后面的任务不会等待
```objective-c
__block int a = 15;
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_async(queue, ^{
  sleep(2);
  a++;
  NSLog(@"block %i", a);
});
sleep(2);
NSLog(@"%i", a);
```
> 常见的网络请求数据多线程执行模型
```objective-c
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
　　//子线程中开始网络请求数据
　　//更新数据模型
　　dispatch_sync(dispatch_get_main_queue(), ^{
　//在主线程中更新UI代码
　});
});
```

#### dispatch_sync（同步）
block提交到队列之后，会先将block执行完，再继续执行queue
```objective-c
__block int a = 15;
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_sync(queue, ^{
  sleep(2);
  a++;
  NSLog(@"block %i", a);
});
sleep(2);
NSLog(@"%i", a);
```
#### dispatch_after
延迟执行block
```objective-c
__block int a = 20;
NSLog(@"%i", a);
//需要用下面的方法初始化 dispatch_time 否则无效
dispatch_time_t delay = dispatch_time(DISPATCH_TIME_NOW, 2*NSEC_PER_SEC);
  dispatch_after(delay, dispatch_get_main_queue(), ^{
    a++;
    NSLog(@"block %i", a);
  });
NSLog(@"%i", a);
```
>block的执行时间由参数 dispatch_time_t when 来确定，与串行队列和并行队列无关，但是内部的执行顺序仍然由队列的类型决定

#### dispatch_apply
重复执行block
```objective-c
dispatch_queue_t newQueue = dispatch_queue_create("com.dispatch.concurrent", DISPATCH_QUEUE_CONCURRENT);
dispatch_apply(10, newQueue, ^(size_t time) {
  NSLog(@"c %zu", time);
});
```

#### dispatch_set_target_queue
* 它会把需要执行的任务对象指定到不同的队列中去处理，这个任务对象可以是dispatch队列，
* 也可以是dispatch源。而且这个过程可以是动态的，可以实现队列的动态调度管理等等。
* 比如说有两个队列dispatchA和dispatchB，这时把dispatchA指派到dispatchB：

## Dispatch Groups？？使用组的意义
### dispatch_group_create
创建组
```objective-c
dispatch_group_async(dispatch_group_create(), dispatch_get_main_queue(), ^{
NSLog(@"block");
});
NSLog(@"---");
```
### dispatch_group_notify
所有block执行完了 在执行notify的block
```objective-c
dispatch_group_t group = dispatch_group_create();//创建一个group

dispatch_group_async(group, dispatch_get_main_queue(), ^{
  NSLog(@"block");
});
dispatch_group_async(group, dispatch_get_main_queue(), ^{
  NSLog(@"block");
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{//所有的block执行完毕之后调用该block
  NSLog(@"notif");
});
dispatch_group_async(group, dispatch_get_main_queue(), ^{
  NSLog(@"block");
});
```
### dispatch_group_wait
延迟执行block
```objective-c
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, dispatch_get_main_queue(), ^{
  NSLog(@"block");
});

dispatch_time_t timeOut = dispatch_time(DISPATCH_TIME_NOW, 3 * NSEC_PER_SEC);
NSLog(@"%ld", dispatch_group_wait(group, timeOut));
```

## Dispatch Barrier
barrier 能够在一个并行的派遣队列中创建一个串行的点。当遇到一个barrier 时，并行队列会先执行完所有在barrier 之前就已经提交的block 然后再执行barrier 的block 然后，执行完成之后，队列重新唤起正常的执行行为
### dispatch_barrier_async
会直接在 唤起 执行barrier 之后继续执行后面的block
```objective-c
dispatch_barrier_async(queue, ^{
  NSLog(@"barrier");
});
```
### dispatch_barrier_sync
会在执行完 barrier 的block 之后才继续执行 后面的 block
```objective-c
dispatch_barrier_sync(queue, ^{
  NSLog(@"barrier");
});
```
>和dispatch_async || dispatch_sync区别??

## Dispatch Source
### 一般使用流程
- 1 创建diapatch_source_t 对象
- 2 添加事件
- 3 因为现在的source是repend的 需要resume
### DISPATCH_SOURCE_TYPE_TIMER
```objective-c
self.timerSource =  dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_main_queue());
dispatch_source_set_timer(self.timerSource, dispatch_time(DISPATCH_TIME_NOW, 0), 1 * NSEC_PER_SEC,  0*NSEC_PER_SEC);
dispatch_source_set_event_handler(self.timerSource, ^{
  NSLog(@"timer block");
});
dispatch_resume(self.timerSource);
```
> 定时器必须是全局的，如果是局部变量的话不会起作用

### DISPATCH_SOURCE_TYPE_DATA_ADD
```objective-c
dispatch_source_t  source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, dispatch_get_main_queue());
dispatch_source_set_event_handler(source, ^{
  NSLog(@"source data %lu",dispatch_source_get_data(source));
});
dispatch_queue_t queue =    dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(5, queue, ^(size_t index) {
  long datum = random()%100;
  NSLog(@"add data %ld", datum);
  dispatch_source_merge_data(source, datum);
});
dispatch_resume(source);
```
