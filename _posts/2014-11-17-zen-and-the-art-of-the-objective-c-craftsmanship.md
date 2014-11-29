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


在`UIViewController`子类的情况下，重写`init`方法是错误的，在调用者调用`initWithNib:bundle:`来初始化你的类时，你的实现将不被调用。这也违反了调用任何指定初始化都应当是合法的规则。

在你自己提供自己的指定初始化方法时，你需要遵循以下三步来保证行为正确。

1. 声明你的指定初始化方法，调用你的直接父类的指定初始化方法
2. 重写你的直接父类的指定初始化方法，调用你自己新加的指定初始化方法
3. 为新加的指定初始化方法添加文档

很多开发者缺少后面两步，这不仅仅是一个小的谨慎点，在这两步的情况下而且是违背框架约定的，并且能导致非常诡异的行为以及错误。我们看一个正确实现这个的示例

```
@implementation ZOCNewsViewController

- (id)initWithNews:(ZOCNews *)news
{
    // call to the immediate superclass's designated initializer
    self = [super initWithNibName:nil bundle:nil];
    if (self) {
        _news = news;
    }
    return self;
}

// Override the immediate superclass's designated initializer
- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    // call the new designated initializer
    return [self initWithNews:nil];
}

@end
```

在你不重写`initWithNibName:bundle:`时，调用者决定用这个方法来初始化你的类（者是一个完全合法的选项），`initWithNews:`方法永远不会被调用到，这将导致不正确的初始化序列，你的特殊的初始化逻辑没有被执行。

即使推断是哪个方法是指定初始化方法是可能的，清晰明确的指出总是好的（将来你或者其他开发者会感谢你的）。有两个策略你可以使用（不是相互排斥的）：第一条是明确的在文档中列出哪个是指定初始化方法，但是更好的也对编译器更加友好的是使用编译器命令`__attribute__((objc_designated_initializer))`,借此你就可以表明你的用意。

使用命令将使得编译器可以帮助你，并且实际上如果你的心得指定初始化方法中不调用父类的指定初始化方法，编译器会给出警告。

有一些情况，不调用该类的指定初始化方法，而调用类树中的其他指定初始化方法将使得类不能工作。对照上面的例子来说，实例化一个应当展示新闻但不好喊news的`ZOCNewsViewController`对象是没有意义的。在这种情况下，你可以强迫必须调用特别的指定初始化方法，简单的使其他指定初始化方法不可用。这可以通过另外一个编译器指令`__attribute__((unavailable(“Invoke the designated initializer”)))`来实现，用该指令装饰一个方法会导致如果你调用这个方法编译器会产生一条错误。

这是上面示例的相关实现（注意宏的实现避免代码的重复以及减少繁琐）

```
@interface ZOCNewsViewController : UIViewController

- (instancetype)initWithNews:(ZOCNews *)news ZOC_DESIGNATED_INITIALIZER;
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil ZOC_UNAVAILABLE_INSTEAD(initWithNews:);
- (instancetype)init ZOC_UNAVAILABLE_INSTEAD(initWithNews:);

@end
```

上面描述的东西的结论是你应当永远不要在指定初始化方法中调用次级初始化方法（如果次级初始化方法遵循原则，它将调用指定初始化方法）。如果这么做了，调用非常可能触发子类中重写的init方法进而导致无限调用循环。

然而对于前面列出的规则有一个例外是一个对象是否遵循`NSCoding`协议并且通过`initWithCoder:`方法来初始化。

你应当区分父类什么时候遵循`NSCoding`什么时候不遵循。

在之前的实例中，如果你仅仅调用`[super initWithCoder:]`,你讲可能和指定初始化方法有一些共同的初始化代码。有个好的处理这种情况的方法是将这些代码提取到一个私有方法(例如：`p_commonInit`)

当你的父类不遵循`NSCoding`协议时，推荐的做法是将`initWithCoder:`按照次级初始化方法来对待，因此调用`self`的指定初始化方法。注意这个是于苹果在[归档和序列化程序指南](https://developer.apple.com/library/mac/documentation/cocoa/Conceptual/Archiving/Articles/codingobjects.html#//apple_ref/doc/uid/20000948-BCIHBJDE)中列出的相违背的。

> 对象应当首先调用其父类的指定初始化方法来初始化继承的状态

遵循这个实际上将导致如果你的类是或不是`NSObject`的子类将会有不确定的行为。

##次级初始化

如上面段落列出的，次级初始化方法是一组比较方便的为指定初始化方法提供默认值或者行为的的方法。这就是说，你不应当在这些方法中做强制的初始化并且从来不应假定这个方法将被调用看起来很清晰。唯一的我们保证被调用的是指定初始化方法。

这意味这在次级初始化方法中你应当总是调用其他的次级初始化方法或者`self`指定初始化方法。有时，由于错误，有人可能会写成`super`；如果这样做将引起前面的初始化序列没有被遵守（在这的特定情况是跳过当前类的初始化）。

参考：

1. [https://developer.apple.com/library/ios/Documentation/General/Conceptual/DevPedia-CocoaCore/ObjectCreation.html](https://developer.apple.com/library/ios/Documentation/General/Conceptual/DevPedia-CocoaCore/ObjectCreation.html)
2. [https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Initialization/Initialization.html](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Initialization/Initialization.html)
3. [https://developer.apple.com/library/ios/Documentation/General/Conceptual/DevPedia-CocoaCore/MultipleInitializers.html](https://developer.apple.com/library/ios/Documentation/General/Conceptual/DevPedia-CocoaCore/MultipleInitializers.html)
4. [https://blog.twitter.com/2014/how-to-objective-c-initializer-patterns](https://blog.twitter.com/2014/how-to-objective-c-initializer-patterns)

#instancetype

有人可能经常没有发现Cocoa中充满了惯例，并且这些惯例使得编译器更加智能。编译器知道是否遇到`alloc`或者`init`方法，即使两个方法返回的类型是`id`，这些方法将返回被调用类的类型的实例对象。因此，这允许编译器进行强制类型检查（例如：检查被调用方法的返回值是合法的）。Clang这种贴心的功能源于被称作[相关结果类型](http://clang.llvm.org/docs/LanguageExtensions.html#related-result-types),具体含义如下：

> 发送alloc和inits等的消息与接收类的实例对象的消息有相同的静态类型

想要了解更多允许自动识别相关结果类型的惯例，请参考Clang语言扩展指南的相关章节。

相关结果类型可以用`instancetype`作为返回结果类型明确列出，并且在工厂方法或者便利构造器被使用的情况下是非常有用的。这可以提示编译器进行正确的类型检查，尤为重要的是，当进行子类化的时候依然可以表现正确。

```
@interface ZOCPerson
+ (instancetype)personWithName:(NSString *)name;
@end
```

虽然，根据clang的说明，`id`可以被编译器升为`instancetype`。在`alloc`或者`init`的情况下，我们强烈鼓励对所有返回类实例对象的类和实例方法使用`instancetype`返回值。

这主要是在所有API中形成习惯以及保持一致（也许有一个更易读的接口）。再次，对代码进行小小的调整就可以提高可读性:简单瞥一眼就能知道哪些方法返回实例对象。长远来看，你将感激这些细节的归类。

参考：
1. [http://tewha.net/2013/02/why-you-should-use-instancetype-instead-of-id](http://tewha.net/2013/02/why-you-should-use-instancetype-instead-of-id/)
2. [http://tewha.net/2013/01/when-is-id-promoted-to-instancetype/](http://tewha.net/2013/01/when-is-id-promoted-to-instancetype/)
3. [http://clang.llvm.org/docs/LanguageExtensions.html#related-result-types](http://clang.llvm.org/docs/LanguageExtensions.html#related-result-types)
4. [http://nshipster.com/instancetype/](http://nshipster.com/instancetype/)

#初始化模式

##类簇

苹果文档中类簇的描述如下：

> 一种将许多私有的，具体的类归类在一个共有的，抽象的父类之下的结构

如果这个描述听起来很熟悉，那么可能你的直觉是正确的。类簇是苹果对[抽象工厂](http://en.wikipedia.org/wiki/Abstract_factory_pattern)设计模式的术语。类簇的想法非常简单：你通常有一个抽象类，该类在初始化过程中使用一些作为初始化方法参数的信息，或者环境中可得到的信息来完成一个逻辑并且实例化一个具体的子类。这个“公共接口”的类内部应当对其子类有很好的了解，以便能够对任务来说最恰当的私有子类。这个模式非常有用，因为它对于调用者去除了初始化逻辑中的复杂性，调用者仅仅知道相互通信的接口，并且也不关心下面的具体实现。

类簇在苹果的框架中广泛使用；最瞩目的例子是`NSNumber`，它可以根据提供数字的类型（integer，float等）返回恰当的子类；或者是`NSArray`， 它根据最优的存储策略返回具体的子类。


这个模式的美好之处在于调用者完全不知道具体子类；实际上，这可以在设计库时使用，只要遵循抽象类中建立的约定，就能交换底层返回类而不暴露任何实现细节。

以我们的经验，类簇在去除条件代码上非常有帮助。一个典型的例子是当你在iPhone和iPad有相同的UIViewController的子类，但是在设备上行为有些轻微的却别。比较天真的实现是在需要不同逻辑方法中放置一些检查设备的代码，即使开始时，应用的条件逻辑非常少，但是他们会自然增长进而产生爆炸式的代码路径。更好的设计可以通过创建一个抽象通用的view controller，该类中包含所有共享的逻辑，然后两个每种设备特殊的子类来达到。

通用的view controller 将检查现在的设备类型，并根据该值返回恰当的子类。

```
@implementation ZOCKintsugiPhotoViewController
- (id)initWithPhotos:(NSArray *)photos
{
    if ([self isMemberOfClass:ZOCKintsugiPhotoViewController.class]) {
        self = nil;
        if ([UIDevice isPad]) {
            self = [[ZOCKintsugiPhotoViewController_iPad alloc] initWithPhotos:photos];
        }
        else {
            self = [[ZOCKintsugiPhotoViewController_iPhone alloc] initWithPhotos:photos];
        }
        return self;
    }
    return [super initWithNibName:nil bundle:nil];
}
@end
```
上面的示例展示了怎么创建类簇。首先`[self isMemberOfClass:ZOCKintsugiPhotoViewController.class]`被用来阻止在子类中重写init方法的必要性，为了阻止无限循环。当`[[ZOCKintsugiPhotoViewController alloc] initWithPhotos:photos]`被调用时，前面的检查逻辑为真，`self = nil`用来去除对`ZOCKintsugiPhotoViewController`实例对象的引用，改对象将被销毁，后面的是选择哪个子类应当被初始化的逻辑。

我们假定在iPhone上运行代码，并且`ZOCKintsugiPhotoViewController_iPhone`没有重写`initWithPhotos`,在这种情况下，当执行`self = [[ZOCKintsugiPhotoViewController_iPhone alloc] initWithPhotos:photos];`时，`ZOCKintsugiPhotoViewController`将被调用，这块，在第一步检查时，对象不是`ZOCKintsugiPhotoViewController`,检查返回失败，然后调用`return [super initWithNibName:nil bundle:nil];`,这使得整个初始化沿着之前章节强调的正确初始化路径进行。

##单例

通常尽可能避免使用单例，而是使用依赖注入。不过，不可避免的单例对象应当使用线程安全的创建共享对象的模式。自GCD起,可以使用`dispatch_once()`来达到目的

```
+ (instancetype)sharedInstance
{
   static id sharedInstance = nil;
   static dispatch_once_t onceToken = 0;
   dispatch_once(&onceToken, ^{
      sharedInstance = [[self alloc] init];
   });
   return sharedInstance;
}
```

dispatch_once()的使用是同步的，替代了底下废弃的惯用方法：

```
+ (instancetype)sharedInstance
{
    static id sharedInstance;
    @synchronized(self) {
        if (sharedInstance == nil) {
            sharedInstance = [[MyClass alloc] init];
        }
    }
    return sharedInstance;
}
```

`dispatch_once()`相比上面的好处是比较快，语义上更加清晰，因为`dispatch_once()`所有的含义是“执行某件事情一次，而且仅一次”,这正精确描述了我们正在做的。这个方法也避免[偶尔可能的奇奇怪怪的崩溃](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html)

经典的可接受的单例对象是GPS和设备加速器。即使单例对象可以子类化，这样使用的也是非常稀少的。接口中应当明显标示这个类将被用作单例。因此，通常一个公开的`sharedInstance`就够了，没有可写的属性被暴露。

尝试使用单例作为对象的容器在你的程序代码中或层次中共享是非常丑陋，令人不爽的，应当被视为设计“臭味”。

#属性

属性应当被命名为尽可能描述性的，避免所写，驼峰结构，开头字母小写。非常幸运，我们选择的工具几乎能补全我们输入的任何东西（看看 Xcode’s Derived Data）,所以，没有原因省下那么点字符，并且，在你的源码中传达尽可能多的信息是更好的。

例如

`NSString *text;`

不要写成

```
NSString* text;
NSString * text;
```
（注解：这种优劣选择对于常量来说是不同的。实际上，这大约是共有的观念以及可读性方面的东西：C++ 开发者更偏向于江变量名与类型分开，这里的话，由于纯粹格式的类型是NSString*（对于分配在堆上的对象，在C++中可能分配在栈上），将使用`NSString* text`）

使用自动生成属性而不是手动用`@synthesize`语句，除非你的属性是协议而不是具体类的一部分。如果XCode能够生成变量，就用它的；此外，这是多余的一部分代码，然而你却必须维护它。

除了在`init`和`dealloc`中，你总是应该使用 getter 和 setter 来获取属性。一般而言，使用属性，你能在视觉上有一些线索，你正在获取的对象是在你的当前作用域之外，因此受副作用影响。

你影响总是选择 setter,原因如下：

* 使用 setter 将遵循预定义的内存管理语法（`strong`,`weak`,`copy`等）。这些定义在ARC 之前是更加相关的，但现在仍相关；想一下这个例子: `copy`语法，每次你使用setter，不用其他任何操作传递的值被拷贝一份。
* KVO 通知将自动触发（`willChangeValueForKey`和`didChangeValueForKey`）
* 更容易调试，你可以在属性声明处设置断点，该断点每次在调用getter，setter时会被触发。或者你可以在自定义的getter/setter里面设置断点。
* 允许在设置值时单一处添加额外的逻辑

你应当总是用 getter

* 对于将来的修改更有弹性(例如：属性是动态生成的)
* 允许子类化
* 容易debug（例如，可以在getter中设置断点来看谁在获取检测的getter）
* 使得意图更加明显：获取ivar `_anIvar`实际上你是在获取`self->_anIvar`。在block内部获取iVar时这可能会导致问题（你讲捕获并保留sef，虽然你没有明确的看到`self`关键字）
* 自动触发KVO通知
* 消息发送带来的效率耗费是非常少的，大多数情况下是可以忽略的。关于属性性能的更多信息你可以找到一个有趣的性能刚要[我应当使用属性还是实例变量](http://blog.bignerdranch.com/4005-should-i-use-a-property-or-an-instance-variable/)


## init和Dealloc

这儿以一个礼物外需要提前列出：你永远不要再init中使用getter或者setter（其他init方法也是如此），替代的你应当总是直接通过实例变量获取变量。这是对子类化的一些防范措施：最终子类可以重写getter或者setter方法，并可能调用其他方法，其他方法中获取的属性或者iVars可能是不确定状态的或者没有全部初始化的。记住一个对象在init返回后才被认为是完全初始化。这些同样在`dealloc`中适用（在`dealloc`中，对象可能在不确定状态）。这之前也被清楚的列出过好多次：

* [高级内存管理指南](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6)在self解释说明章节"不要使用在初始化或者销毁时获取方法"
* [迁移到现代objective-c](http://adcdownload.apple.com//wwdc_2012/wwdc_2012_session_pdfs/session_413__migrating_to_modern_objectivec.pdf) WWDC 2012 27幻灯片
* Dave 的 [合并请求](https://github.com/NYTimes/objective-c-style-guide/issues/6)

而且，在init中使用init对于`UIAppearence`代理来说不友好。（请参考[自定义视图使用UIAppearence](http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/)）

### 点符号

使用setter/getter时，经常选用点符号。获取改变属性时应当总是使用点符号。

例如：

```
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

不要如下

```
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

使用点符号将有助于视觉上区分属性以及典型的方法调用。


## 属性声明

更好的声明属性的格式如下：

```
@property (nonatomic, readwrite, copy) NSString *name;
```

属性的属性应当按照以下顺序排列:原子化, 读写和内存管理存储。这么做的话，你的属性更可能在正确位置更改并且容易用眼睛扫描。

除非你有严格的必要性，你一定要使用`nonatomic`属性。在iOS上，`atomic`引入的锁会显著的影响性能。

存储block的属性，为了在声明作用域结束后依然使block存活，必须使用`must`。（block初始时在栈上创建，调用copy使得block被拷贝到堆上）.

为了得到一个共有的getter以及私有的setter，你可以用`readonly`声明共有属性，然后再类的扩展中重新用`readwrite`声明同样的属性:

```
@interface MyClass : NSObject
@property (nonatomic, readonly) NSObject *object
@end

@implementation MyClass ()
@property (nonatomic, readwrite, strong) NSObject *object
@end

```

如果`BOOL`类型的属性名字被表述为形容词，该属性可以省略'is'前缀，对get存取方法特别指出惯例名字，例如：

```
@property (assign, getter=isEditable) BOOL editable;
```

示例摘自[cocoa命名指南](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE)

在实现文件中避免使用`@synthesize`,XCode已经添加了这个。

###私有属性

私有属性应当在类的实现文件的类扩展中（匿名分类）声明。 命名分类（`ZOCPrivate`）除了扩展其他类之外不要使用.

例如：

```
@interface ZOCViewController ()
@property (nonatomic, strong) UIView *bannerView;
@end
```

##可变对象

审核可以被设置为可变对象（`NSString`，`NSArray`,`NSURL`）的属性必须设置内存管理类型为`copy`。这样做是为了确保封装性，阻止在设置了属性之后该属性变化但是对象不知道的情况。

你应当避免在公共接口中暴露可变对象，这允许你的类的使用者改变你自己的内部变现以及打破封装性。。你这一提供一个使用只读来返回你对象的不可变拷贝

```
/* .h */
@property (nonatomic, readonly) NSArray *elements

/* .m */
- (NSArray *)elements {
  return [self.mutableElements copy];
}
```
##延迟加载

有些情况如当初始化一个对象很昂贵或者需要配置一次，其中有些配置你不想在调用者的方法中实现。

这种情况下，有人相比于在init方法中创建对象会选择重写属性getter方法以便延迟加载。通常这种操作的模板如下：

```
- (NSDateFormatter *)dateFormatter {
  if (!_dateFormatter) {
    _dateFormatter = [[NSDateFormatter alloc] init];
        NSLocale *enUSPOSIXLocale = [[NSLocale alloc] initWithLocaleIdentifier:@"en_US_POSIX"];
        [dateFormatter setLocale:enUSPOSIXLocale];
        [dateFormatter setDateFormat:@"yyyy-MM-dd'T'HH:mm:ss.SSSSS"];
  }
  return _dateFormatter;
}
```

即使这种操作在某些环境中是有益的但是我们建议当这么做时仔细思量，并且实际上这样的行为应当避免。下面的是反对在属性getter方法使用延迟加载的理由：

* getter方法不应当有副作用。看到赋值右边你不会想到这会引起创建一个对象或者导致副作用。实际上，在不使用getter返回值时调用getter编译器会产生警告“getter不应当用来触发副作用”
* 你把初始化时的消耗作为副作用移到了第一次获取里，这导致很难优化性能问题（这也是非常难用工具进行的）
* 初始化的时机不确定：例如你期望这个属性在一个方法中第一个被获取，但是你一旦改变了类的实现，访问器在你原来期望之前被调用。这可能会导致问题，尤其是在初始化逻辑依赖类中其他可能不同的状态时。通常明确表明这种依赖是更好的做法。
* 这种行为可能不是KVO友好的。如果getter方法改变了对象指向，他应当触发KVO通知来通知该变化，在调用一个getter方法时受到变化通知是非常诡异的。

#方法

##参数断言

你的方法可能需要一些满足一些条件的参数（如不能是nil）：在这些情况下，断言条件并且甚至抛出异常是好的实践。

##私有方法

永远不要给私有方法中添加单个下划线`_`，这个前缀被苹果保留，如果这么做了，你可能冒着覆盖已经存在的苹果的私有方法的分享。

##相等


你需要实现相等时请记住约定：你需要同时实现`isEqual`和`hash`两个方法。如果两个对象对于`isEqual`被认为是相等的，那么`hash`方法必须返回相等的值，但是如果`hash`返回相等的值，对象不一定保证相等。

这个约定归结为存储在收集器中的对象怎么做查找（即：`NSDictionary`和`NSSet`底层使用hash表数据结构）

```
@implementation ZOCPerson

- (BOOL)isEqual:(id)object {
    if (self == object) {
        return YES;
    }

    if (![object isKindOfClass:[ZOCPerson class]]) {
        return NO;
    }

    // check objects properties (name and birthday) for equality
    ...
    return propertiesMatch;
}

- (NSUInteger)hash {
    return [self.name hash] ^ [self.birthday hash];
}

@end
```

需要注意的是hash方法不可以返回常量。这是典型的错误，并且会引起错误，这将一起hash表中100%的冲突，由于hash表中使用hash方法返回的值作为关键字。


















