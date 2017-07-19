---
title: Singleton of iOS
date: 2016-07-10 16:27:13
thumbnail: /images/Singleton/Singleton_2.png
categories:
- iOS
tags:
- iOS
- Singleton
---
# 单例模式 － Singleton
单例模式确保每个指定的类只存在一个实例对象，并且可以全局访问那个实例

## 产生的背景／解决的问题
- 某个类经常被使用，避免其被频繁的创建销毁
- 共享数据，如NSUserDefault进行本地化
![单例使用原理](/images/Singleton/Singleton_1.png)

## 主要结构

![单例的结构](/images/Singleton/Singleton_2.png)
> 这是一个日志类，有一个属性 (是一个单例对象) 和两个方法 (sharedInstance() 和 init())。第一次调用 sharedInstance() 的时候，instance 属性还没有初始化。所以我们要创建一个新实例并且返回。下一次你再调用 sharedInstance() 的时候，instance 已经初始化完成，直接返回即可。这个逻辑确保了这个类只存在一个实例对象。

## 实现方式

**下面是[Apple官方文档里面关于单例(Singleton)的示范代码](https://developer.apple.com/legacy/library/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaObjects/CocoaObjects.html#//apple_ref/doc/uid/TP40002974-CH4-SW32)，MRC。**线程安全的问题，这里先不说。

```objectivec
static MyGizmoClass *sharedGizmoManager = nil;

+ (MyGizmoClass*)sharedManager {
  if (sharedGizmoManager == nil) {
  sharedGizmoManager = [[super allocWithZone:NULL] init];
  }
  return sharedGizmoManager;
}

+ (id)allocWithZone:(NSZone *)zone {
  return [[self sharedManager] retain];
}

- (id)copyWithZone:(NSZone *)zone {
  return self;
}

- (id)retain {
  return self;
}

- (NSUInteger)retainCount {
  return NSUIntegerMax;  //denotes an object that cannot be released
}

- (void)release {
  //do nothing
}

- (id)autorelease {
  return self;
}
```

### 步骤
1. 为单例对象创建一个静态实例，可以写成全局的，也可以在类方法里面实现，并初始化为`nil`；
2. 实现一个实例构造方法，检查上面声明的静态实例是否为`nil`，如果是，则创建并返回一个本类的实例；
3. 重写`allocWithZone方法`，用来保证其他人直接使用`alloc`和`init`试图获得一个新实力的时候不产生一个新实例；
4. 适当实现`copyWithZone`，`mutableCopyWithZone`，非arc下还需要实现`release`和`autorelease`方法。

### 说明
#### static变量
>  从面向对象的角度触发，当需要一个数据对象为**整类而非某个对象服务**，同时有力求不破坏类的封装性，既要求此成员隐藏在类的内部，又要求对外不可见的时候，就可以使用static。

- 这个变量在编译期会对这个变量进行初始化赋值，也就是说这个变量值要么为nil，要么在编译期就可以确定其值，一般情况下，只能用NSString或者基本类型;
- `static`变量存储在静态内存中，不属于堆和栈，程序只会分配一次内存，会在程序退出的时候释放。

####  Alloc && AllocWithZone
- 这两个方法是控制内存分配的；
- 在`NSObject`这个类的[官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/index.html)里面，`allocWithZone`方法介绍说，该方法的参数是被忽略的，正确的做法是传nil或者NULL参数给它。而这个方法之所以存在，是历史遗留原因;
- `[[self alloc] init]`调用时，会默认调用`+(id)allocWithZone:(NSZone *)zone`(这里不讨论什么是`NSZone`)，为了保持单例类实例的唯一性，需要覆盖所有会生成新的实例的方法，如果有人初始化这个单例类的时候不走`[[Class alloc] init]`，而是直接`allocWithZone`，那么这个单例就不再是单例了，所以必须把这个方法也堵上；

#### copyWithZone && mutableCopyWithZone
- 为了避免在`copy`的时候创建一个新的对象，我们需要重写`copyWithZone`;

#### 线程安全
![线程安全示意图](/images/Singleton/Singleton_3.png)

- 为了避免出现上图所示的问题，这里我们可以在init的时候加上线程锁；

```objectivec
+ (instancetype)sharedInstance {
  static MySingleton *_singleton = nil;
  @synchronized(self) {
    if (!_singleton) {
      _singleton = [[self alloc] init];
    }
  }
  return _singleton;
}

```

### 最终实现代码(ARC)
#### 依据上面的说明我们来实现一个单例

``` objectivec
+ (instancetype)sharedInstance {
  @synchronized(self) {
    if (!_instance) {
      _instance = [[self alloc] init];
    }
  }
  return _instance;
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
  return [self sharedInstance];
}

- (id)copyWithZone:(NSZone *)zone {
  return _instance;
}
```

#### 我们还可以通过GCD来实现单例

```objectivec
static GCDSingleton *_instance = nil;

+ (instancetype)sharedInstance {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
  _instance = [[self alloc] init];
  });
  return _instance;
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    _instance = [super allocWithZone:zone];
  });
  return _instance;
}

- (id)copyWithZone:(NSZone *)zone {
  return _instance;
}
```

#### 这里我们介绍一种创建单例的宏(看起来用的很爽，反正我不推荐)

```objectivec
// .h文件
#define CHSingletonH(name) + (instancetype)shared##name;

// .m文件
#define CHSingletonM(name) \
static id _instance; \
\
+ (instancetype)allocWithZone:(struct _NSZone *)zone \
{ \
  static dispatch_once_t onceToken; \
  dispatch_once(&onceToken, ^{ \
  _instance = [super allocWithZone:zone]; \
  }); \
    return _instance; \
  } \
\
+ (instancetype)shared##name \
{ \
  static dispatch_once_t onceToken; \
  dispatch_once(&onceToken, ^{ \
    _instance = [[self alloc] init]; \
    }); \
    return _instance; \
} \
\
- (id)copyWithZone:(NSZone *)zone \
{ \
  return _instance; \
}
```
***====================用法=====================***

```objectivec
@interface MacroSingleton : NSObject
CHSingletonH(MacroSingleton)
@end

@implementation MacroSingleton
CHSingletonM(MacroSingleton)
@end
```
**注意**：用的时候后面没有`";"`

#### 偷懒的方法有很多种，比如下面
Xcode4 引入了一个新`feature`: `code snippets`，具体用法自行查找。下面提供两篇博客的链接

- [使用 Git 来管理 Xcode 中的代码片段](http://blog.devtang.com/2012/02/04/use-git-to-manage-code-snippets/)
- [Xcode Snippets](http://nshipster.cn/xcode-snippets/)


### Swift实现单例的进化之路
#### 先仿照OC的实现方式进行，相传这是最丑的方法

```swift
class SwiftSingleton: NSObject {
class var sharedInstance: SwiftSingleton {
  struct Static {
    static var onceToken: dispatch_once_t = 0
    static var instance: SwiftSingleton? = nil
  }
  dispatch_once(&Static.onceToken) {
    Static.instance = SwiftSingleton()
  }
  return Static.instance!
  }
}
```

> 因为 Swift 1.2 之前并不支持存储类型的类属性，所以我们需要使用一个 struct 来存储类型变量。

#### Swift 里用 `let` 保证线程安全
```swift
class SwiftSingleton {
  class var sharedManager : SwiftSingleton {
  struct Static {
    static let sharedInstance : SwiftSingleton = MyManager()
    }
    return Static.sharedInstance
  }
}
```

#### Swift 1.2 之前的最佳实践
由于 Swift 1.2 之前 class 不支持存储式的 property，我们想要使用一个只存在一份的属性时，就只能将其定义在全局的 scope 中。值得庆幸的是，在 Swift 中是有访问级别的控制的，我们可以在变量定义前面加上 private 关键字，使这个变量只在当前文件中可以被访问。这样我们就可以写出一个没有嵌套的，语法上也更简单好看的单例了：

```swift
private let sharedInstance = MyManager()

class MyManager  {
  class var sharedManager : MyManager {
    return sharedInstance
  }
}
```

#### Swift 1.2 之后的推荐用法
```
class SwiftSingleton  {
  static let sharedInstance = SwiftSingleton()
  private init() {}
}
```
在初始化类变量的时候，Apple 将会把这个初始化包装在一次 swift_once_block_invoke 中，以保证它的唯一性。另外，我们在这个类型中加入了一个私有的初始化方法，来覆盖默认的公开初始化方法，这让项目中的其他地方不能够通过 init 来生成自己的 MyManager 实例，也保证了类型单例的唯一性
## 优缺点

### 优点
- 提供了对唯一实例的受控访问。
- 由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象单例模式无疑可以提高系统的性能。
- 因为单例模式的类控制了实例化的过程，所以类可以更加灵活修改实例化过程。

### 缺点
- 由于单利模式中没有抽象层，因此单例类的扩展有很大的困难。
- 单例类的职责过重，在一定程度上违背了“单一职责原则”。

## 注意事项；
- 不要继承单例类
- 先创建子类永远是子类对象
- 先创建父类永远是父类对象

## 应用实例
- 应用本身就是一个单例`[UIApplication sharedApplication]`

```
UIWindow *window = [UIApplication sharedApplication].keyWindow;
UINavigationController * navigation = (UINavigationController *)window.rootViewController;
```

- 我们常用`NSUserDefaults`本地化一些数据

```objectivec
NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
[userDefaults setObject:obj forKey:key];
[userDefaults objectForKey:key];
```

- 网络请求在我们的应用开发中我觉得是最常用的需要代码复用的部分，因为他使用次数多，且代码都差不多，不受别的模块代码干扰，即低耦合,高内聚。我们进行网络请求要先创建一个`manager`，就是说每次请求我们都需要创建`manager`，但是频繁的创建销毁很浪费系统资源，完全没有必要，因此我们就可以将`manager`封装成一个单例。在[AFNetworkingDemo](https://github.com/AFNetworking/AFNetworking)中就是将`AFHTTPSessionManager`封装成一个单例。

```objectivec
+ (instancetype)sharedClient {
  static AFAppDotNetAPIClient *_sharedClient = nil;
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
  _sharedClient = [[AFAppDotNetAPIClient alloc] initWithBaseURL:[NSURL URLWithString:AFAppDotNetAPIBaseURLString]];
  _sharedClient.securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeNone];
  });

  return _sharedClient;
}
```
- `[NSNotificationCenter defaultCenter]`
