---
layout: post
title: objective-c中的禅与艺术
category: articles
---


iOS-blog [source page](http://ios-blog.co.uk/resources/free-pdf-zen-and-the-art-of-the-objective-c-craftsmanship/)

#前言
我们2013年11月开始写这本书。最初的目标是为写出尽可能最简洁objective-c代码提供指南（现在有太多的指南，并且所有的都是不尽人意的）。我们不打算介绍硬性的规则，取而代之的是一种在开发者中尽可能统一的写代码方式。随着时间的推移，我们的视线转移到解释如何设计组织良好代码上面。

以下想法是这样的，代码不应当是仅仅为了编译，更要是“经得起检验”的。好的代码有几个特征：简洁，自解释，良好组织，良好的文档，好的命名，经过好的设计以及经得起时间考验。

幕后的主要目标是清晰总能够取得性能的提升并且每一个选择总是应当提供相应的原理。这里讨论的一些话题是通用的，独立于语言的即使一切都是捆绑在objective-c上

#swift

2014年6月6日，苹果宣布一门新的编程语言将被用在iOS和mac开发商：swift。这门新的语言完全不同于objective-c，当然引起了我们写这本书计划的改变。最终变成将这篇文章的现状发布出来，并且不在继续展开我们之前计划包含的话题。

objective-c 将不在改进，与此同时，继续写一本关于这们语言的数将不会获得同于往日的关注，当然也不是一个聪明的选择。

可以看看这些非常棒的[iOS8 swift教程](http://ios-blog.co.uk/swift-tutorials/)

#聊聊社区
我们已经免费的，面向社区的发布了这本书，因为我们希望对读者带来价值，如果你们任何一个人能从这里学习到一条最佳实践那么我们的目标就达到了。

我们已经竭尽所能润色文字并且令读者更易阅读，但是我们可能留下错别字，错误，不完整。我们极力鼓励你给我们反馈以及改进。所以如果有的话请联系我们。我们期待代码合并请求。

#Authors

##Luca Bernardi

* http://lucabernardi.com
* @luka_bernardi
* http://github.com/lukabernardi

##Alberto De Bortoli

* http://albertodebortoli.com
* @albertodebo
* http://github.com/albertodebortoli

#条件句

条件句应当总是使用花括号以避免错误，即使条件主体可以不用花括号来写（如，仅仅一行）。这些错误包括新增一行并且期望它使条件句的一部分。另外一个甚至更加危险的不足可能出现在条件句的原有条件逻辑被注释掉，下一行代码不知不觉就成为了条件语句的一部分。

优选的：

```
if (!error) {
    return success;
}
```

不推荐的

```
if (!error)
    return success;
```

或者

```
if (!error) return success;
```

在2014年2月广为人知的[goto fail](https://gotofail.com/)在苹果的SSL/TLS实现中被发现。这个bug就是因为在if条件后一个if条件后的重复goto语句。用括号包起if分支就可以阻止这种问题发生。

代码摘录如下

```
static OSStatus
SSLVerifySignedServerKeyExchange(SSLContext *ctx, bool isRsa, SSLBuffer signedParams,
                                 uint8_t *signature, UInt16 signatureLen)
                                 
{
  OSStatus        err;
  ...
  if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
    goto fail;
  if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
    goto fail;
    goto fail;
  if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
    goto fail;
  ...
fail:
  SSLFreeBuffer(&signedHashes);
  SSLFreeBuffer(&hashCtx);
  return err;
}
```

很容易找到有两行```goto fail;```，一个在一个下面，没有括号。我们很清楚不想冒险制造出像上面这个一样的bug，对吗？

另外，这种风格对所有条件语句是更加一致的，并且更加容易扫描。

#yoda 条件

总是避免使用Yoda条件。yoda条件是比较常量与一个变量而不是其他的方法。这就像说“如果蓝色是天空”或者“如果高的是那个人”而不是说“天空是蓝色的”，“那个人是高的”

优选的

```
if ([myValue isEqual:@42]) { ...
```

不推荐的

```
if ([@42 isEqual:myValue]) { ...
```

#nil和BOOL检查

与Yoda条件相同，nil检查也在争论的核心。一些书本像下面一样检查一个对象是否为nil

```
if (nil == myValue) { ...
```

有人可能会争论假如将nil看做常量，这是不恰当或者与Yoda条件相似的。这样写的原因是一些程序员使用这种方法来避免一些难以调试的bug。考虑一下下面的代码：

```
if (myValue == nil) { ...
```

如果发生写错的情况，程序员可能实际写成：

```
if (myValue = nil) { ...
```

这条语句是一条合法的赋值语句，即使你是一个经验丰富的程序员（因此可能还有点视力问题）也的确难于调试。这种情况在将nil放在左边时从来不会发生，因为nil不可以被赋值。也有种说法是如果程序员使用了这种写法，他很明确知道这样写的动机，因此十多年来所有的事情是恰好两次检查了切好的输入更好一些。

在此更进一步，为了避免这样的大惊小怪，不留下怀疑的空间，更好方法是使用感叹号。因为nil解析为No，所以在条件里面比较是不必要的。同时从来不要与YES比较，因为YES被定义为1，而BOOL则可以一直到8bits，如char下面的。

优选的

```
if (someObject) { ...
if (![someObject boolValue]) { ...
if (!someObject) { ...
```

不推荐的

```
if (someObject == YES) { ... // Wrong
if (myRawValue == YES) { ... // Never do this.
if ([someObject boolValue] == NO) { ...
```

这个考虑也是为了文件之间的一致性以及视觉上更清晰。

#最优路径

当书写条件相关的代码时，左手边的代码应当是最优路径。这就是不要嵌套if语句。多条return语句是可以接受的。这可以避免嵌套复杂性的增长，并使得代码易读，因为最重要的代码部分没有嵌套在条件分支中，你有视觉上的条理什么是最相关的代码。

优选的

```objc
- (void)someMethod {
  if (![someOther boolValue]) {
      return;
  }
  //Do something important
}
```

不推荐的

```objc
- (void)someMethod {
  if ([someOther boolValue]) {
    //Do something important
  }
}
```

#复杂的条件

如果你在if中有复杂的条件，你应当提取出来赋值到一个BOOL值上使得逻辑以及每一个条件更清楚。

```
BOOL nameContainsSwift  = [sessionName containsString:@"Swift"];
BOOL isCurrentYear      = [sessionDateCompontents year] == 2014;
BOOL isSwiftSession     = nameContainsSwift && isCurrentYear;

if (isSwiftSession) {
    // Do something very cool
}
```

#三元操作符

三元操作符`?`应当仅仅用在增加代码整洁清晰上。一条简单的逻辑通常是所有的需要计算的。计算多个条件，则像if语句以及或者重构成实例变量更容易理解

优选的

```
result = a > b ? x : y;
```

不推荐的

```
result = a > b ? x = c > d ? c : d : y;
```

当三元操作符第二个参数返回条件检查中的同一个对象是，下面的语法简洁：

优选的

```
result = object ? : [self createObject];
```

不推荐的

```
result = object ? object : [self createObject];
```

#错误处理

当方法中返回以引用返回错误，检查返回结果而不是错误值

优选的

```
NSError *error = nil;
if (![self trySomethingWithError:&error]) {
    // Handle Error
}
```

此外，一些苹果的API在成功的情况下将一些不需要的值赋给error参数，所以检查error可能导致漏报（随后而来的crash）。

#case 语句

对于case语句花括号不是必须的，除非编译器强制。当case里面包含不止一行时，花括号应当被加上。

```
switch (condition) {
    case 1:
        // ...
        break;
    case 2: {
        // ...
        // Multi-line example using braces
        break;
       }
    case 3:
        // ...
        break;
    default: 
        // ...
        break;
}
```

有些时候同样的代码被多个case条件使用，这时应当使用fall-through。fall-through 是去除一条case的break语句进而允许进入到下个case的执行流程中。

```
switch (condition) {
    case 1:
    case 2:
        // code executed for values 1 and 2
        break;
    default: 
        // ...
        break;
}
```

当在switch中使用了枚举类型，就不需要`default`。例如

```
switch (menuType) {
    case ZOCEnumNone:
        // ...
        break;
    case ZOCEnumValue1:
        // ...
        break;
    case ZOCEnumValue2:
        // ...
        break;
}
```

而且，不适用default条件，当enum中新增加一个值时，程序员立即就会注意到相应的warning：
`Enumeration value ‘ZOCEnumValue3′ not handled in switch.`

#枚举类型

使用enum时，推荐使用新的固定基础类型规范，因为这会带来更强的类型检查以及代码补全。现在的SDK包含了一个宏帮助鼓励使用固定基础类型-`NS_ENUM()`,例如

```
typedef NS_ENUM(NSUInteger, ZOCMachineState) {
    ZOCMachineStateNone,
    ZOCMachineStateIdle,
    ZOCMachineStateRunning,
    ZOCMachineStatePaused
};
```


#命名

#一般惯例

Apple的命名惯例是任何地方都应当尽可能坚持的，尤其是于[内存管理规则](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html)相关的([NARC](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html))

长的描述型式的方法变量名称是非常好的。

优选的

```
UIButton *settingsButton;
```

不建议的

```
UIButton *setBut;
```

#常量

常量应当遵循所有单词首字母大写的驼峰命名并且为了清晰起见以相关的类名字作为前缀。

优选的

```
static const NSTimeInterval ZOCSignInViewControllerFadeOutAnimationDuration = 0.4;
```

不推荐的

```
static const NSTimeInterval fadeOutTime = 0.4;
```

常量是比直接写字符串或者数字更好的选择，因为这样使得通用变量重复使用变得容易，并且可以不用搜索查找快速改变。常量应当声明为`static`常量而不是`#define` 除非非常明确的作为宏使用。

优选的

```
static NSString * const ZOCCacheControllerDidClearCacheNotification = @"ZOCCacheControllerDidClearCacheNotification";
static const CGFloat ZOCImageThumbnailHeight = 50.0f;
```

不推荐的

```
#define CompanyName @"Apple Inc."
#define magicNumber 42
```

外部使用的常量应当按照下面的样式在头文件中声明

```
extern NSString *const ZOCCacheControllerDidClearCacheNotification;
```

之前的被定义的赋值应当在实现文件中。

你应当为公共的常量添加命名空间。虽然实现文件中使用的常量遵循另外一种模式，没有必要与上面的规则保持一致。

#方法

在方法签名中，在方法类型(`-`/`+`)后面应当有一个空格。方法片段间应当是一个空格。在参数之前总是有描述性的关键字来描述参数。

"and"关键字被保留使用。不应当在多参数中使用，如下面`initWithWidth:height:`示例中展示的一样

优选的

```
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
- (void)sendAction:(SEL)aSelector to:(id)anObject forAllCells:(BOOL)flag;
- (id)viewWithTag:(NSInteger)tag;
- (instancetype)initWithWidth:(CGFloat)width height:(CGFloat)height;
```

不推荐的

```
- (void)setT:(NSString *)text i:(UIImage *)image;
- (void)sendAction:(SEL)aSelector :(id)anObject :(BOOL)flag;
- (id)taggedView:(NSInteger)tag;
- (instancetype)initWithWidth:(CGFloat)width andHeight:(CGFloat)height;
- (instancetype)initWith:(int)width and:(int)height;  // Never do this.
```

#字面值

在创建不可变对象实例时应当使用`NSString`,`NSDictionary`,`NSArray`和`NSNumber`字面值。特别注意传进`NSArray`，`NSDictionary`字面值中的`nil`值，那将引起崩溃。

例如

```
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
```

不要写成

```
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
```

对于可变的我们推荐是使用拷贝，而不要直接使用像`MSMutableArray`，`NSMutableString`等这样的类。

下面的写法应当避免

```
NSMutableArray *aMutableArray = [@[] mutableCopy];
```

上面的写法存在性能以及可读性方面的问题。

性能方面，不变对象被创建而后立刻释放，这种写法大概不会拖慢你的软件（除非频繁调用）但是仅仅为省下一些字符不算原因。

关于可读性方面，这里有两个问题：第一个是扫过代码时看到`@[]`第一个想到的是不变的`NSArray`实例变量，但是在这，比必须停下，更加耐心的检查。另外需要重视的一面是你的代码可能被没有经验的人看到，基于他的背景，他可能对可变对象与不可变对象之间区分不熟。他可能不太明白为什么创建一个可变的拷贝（但是我们并不建议这些知识不应当被掌握）。再次，这虽然不是绝对的错误，但是关乎代码的可用性（包含可读性）。


#类

#类名

你应当总是使你的类以*三个*大写字母为前缀（两个字母是苹果保留的）。这个看起来很怪的实践是缓和我们喜爱的语言命名空间缺失的缺陷。一些开发者在Model对象上并没有遵守这个实践（我们发现在coredata中尤为严重），我们建议在coredata 对象中也严格遵守这样的惯例，因为你可能最终需要合并其他的Managed object Model，也许来自第三方。就像你注意到的，这本书里类是一`ZOC`为前缀。

还有一个在选择类名中好的实践你可能想要遵循：在创建新的类时，将新类特有的名字部分放在前缀与父类名字之间。这个用示例可以更好的解释：如果你有一个类名字`ZOCNetworkClient`，示例的子类名将是`ZOCTwitterNetworkClient`(注意"Twitter"在"ZOC"与“NetworkClient”之间)。或者遵循同样的规则，`UIViewController`的子类是`ZOCTimeLineViewController`。

#初始化和销毁

我们推荐的组织代码的方式是将`dealloc`放在implementation的最前面（直接在`@synthisize`和`@dynamic`之后）,`init`方法直接在`dealloc`之后。多个初始化方法的情况下，指定的初始化方法应当放在前面，因为在它里面可能有最复杂的逻辑，次级的初始化方法排在第二位。在使用ARC的情况下，很少实现dealloc方法，但是dealloc与init放在一块写仍然是基本原则，这样视觉上强调这两个方法这件的匹配。通常dealloc方法中取消的事情是在init中做的事情。

`init`方法应当像下面这样组织

```
- (instancetype)init
{
    self = [super init]; // call the designated initializer
    if (self) {
        // Custom initialization
    }
    return self;
}
```

理解为什么我们需要设置self值为`[super init]`的返回以及如果我们不这样做会有什么影响是非常有趣的。

让我们做一下回溯：我们习惯于写像`[[NSObject alloc] init]`这样的表达式，导致`alloc`与`init`的区别被淡化了。这个objective-c的特性称之为两步创建。这意味着分配（allocation）和初始化(initialization)是分开的两步，因此两个不同的方法需要被调用:`alloc`和`init`。

* `alloc` 负责对象的分配。这个过程涉及从程序的虚拟内存中分配足够的内从来保存对象，赋值`isa`指针，初始化retain count，将所有实例变量置为0.
* `init` 负责初始化对象，这意味着使得对象变的可用。典型的意味着将实例变量设置为需要的可用的初始化值。

`alloc`方法返回一个没有初始化的合法对象。每一个发送给这个对象的方法将被转到`objc_msgSend()`调用，此时`self`参数指向`alloc`返回的对象；这时在每个方法中`self`是隐式可用的。总结两步创建第一个发给新分配的对象惯例上应当是一个`init`方法。很明显，`NSObject`的`init`实现除了返回`self`没有做其他事情。

`init`方法一个重要的约定是可以通过返回`nil`告知调用者初始化没有成功完成;初始化失败的原因很多，例如传进的一个错误的格式或者初始化一个需要的对象失败。

这引导我们理解为什么我们总是调用`self=[super init]`。如果你的父类说明没有成功初始化自己，你最好假设你处在矛盾的状态下并且因此不应当进行你自己的初始化，相反你的实现应当返回`nil`。如果你没有这样做，你最终可能使用一个不可用的对象，这可能导致一些非预期的行为发生，设置导致你的程序崩溃。

从新赋值`self`的能力可以被`init`方法用来返回一个与之前被调用对象（alloc）不同的对象。这种行为的例子是[类簇](http://ios-blog.co.uk/resources/free-pdf-zen-and-the-art-of-the-objective-c-craftsmanship/#class-cluster)或者一些对一些完全相等的对象返回同一个对象的cocoa类。

#指定的和次级初始化

objective-c中又指定与次级初始化的概念。指定的初始化包含全部初始化参数的初始化，次级初始化是一个或者多个调用指定初始化方法，指定一些默认参数值的初始化方法。

```
@implementation ZOCEvent

- (instancetype)initWithTitle:(NSString *)title
                         date:(NSDate *)date
                     location:(CLLocation *)location
{
    self = [super init];
    if (self) {
        _title    = title;
        _date     = date;
        _location = location;
    }
    return self;
}

- (instancetype)initWithTitle:(NSString *)title
                         date:(NSDate *)date
{
    return [self initWithTitle:title date:date location:nil];
}

- (instancetype)initWithTitle:(NSString *)title
{
    return [self initWithTitle:title date:[NSDate date] location:nil];
}

@end
```

上面的事例中`initWithTitle:date:location:`是指定初始化方法，其他的两个初始化是次级初始化，因为他们仅仅调用他们实现类中的指定初始化

##指定初始化

一个类中应当有一个且仅有一个指定初始化方法，其他的init方法应当调用指定初始化（即使这种情况可能有一些例外）。这个区别没有提应当调用哪个初始化方法。调用类集成树中的任何一个指定初始化方法都应当是合法的，并且所有的类继承树中的指定初始化方法应当从最远的祖先（典型的是`NSObject`）开始到你的类被调用是应当保障的。

实践中，这样说的含义是第一个被执行初始化的代码是最远祖先的，然后沿着继承树向下，给继承树种所有类执行他们特殊初始化代码的机会。这所有的意味着你想要从父类继承下来的一切在做实际工作时是可用的。即使这没有明确列出，所有的苹果框架都是保证遵循这条约定，你的类应当同样遵循保证成为一个好"市民"并且表现如预期。

在定义新类是可能出现三种不同情况：

1. 不需要重写初始化
2. 重写指定初始化
3. 定义一个新的指定初始化

第一个不需多说，我们不需要为初始化方法中添加任何特殊逻辑，你只需要依赖父类的指定初始化就好。

当你想为初始化方法提供进一步的逻辑，你可以决定重写指定初始化。你仅仅应当重写直接父类的指定初始化方法并且确保调用super你所重写方法；

一个典型的例子是创建`UIViewController`子类时是否重写`initWithNibName:bunlde`

```
@implementation ZOCViewController

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    // call to the superclass designated initializer
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self) {
        // Custom initialization
    }
    return self;
}

@end
```


在`UIViewController`子类的情况下，重写`init`方法是错误的












