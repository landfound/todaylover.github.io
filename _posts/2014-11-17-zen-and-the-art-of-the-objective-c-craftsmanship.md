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

2014年6月6日，苹果宣布一门新的编程语言将被用在iOS和mac开发上：swift。这门新的语言完全不同于objective-c，当然引起了我们写这本书计划的改变。最终变成将这篇文章的现状发布出来，并且不在继续展开我们之前计划包含的话题。

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

属性应当被命名为尽可能描述性的，避免缩写，驼峰结构，开头字母小写。非常幸运，我们选择的工具几乎能补全我们输入的任何东西（看看 Xcode’s Derived Data）,所以，没有原因省下那么点字符，并且，在你的源码中传达尽可能多的信息是更好的。

例如

`NSString *text;`

不要写成

```
NSString* text;
NSString * text;
```
（注解：这种优劣选择对于常量来说是不同的。实际上，这大约是共有的观念以及可读性方面的东西：C++ 开发者更偏向于将变量名与类型分开，这里的话，由于纯粹格式的类型是NSString*（对于分配在堆上的对象，在C++中可能分配在栈上），将使用`NSString* text`）

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

而且，在init中使用getter/setter对于`UIAppearence`代理来说不友好。（请参考[自定义视图使用UIAppearence](http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/)）

### 点符号

使用setter/getter时，经常选用点符号。获取、改变属性时应当总是使用点符号。

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

任何可以被设置为可变对象（`NSString`，`NSArray`,`NSURL`）的属性必须设置内存管理类型为`copy`。这样做是为了确保封装性，阻止在设置了属性之后该属性变化但是对象不知道的情况。

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


你需要实现相等时请记住约定：你需要同时实现`isEqual`和`hash`两个方法。如果两个对象对于`isEqual`被认为是相等的，那么`hash`方法必须返回相等的值，但是如果`hash`返回相等的值，对象则不一定保证相等。

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

需要注意的是hash方法不可以返回常量。这是典型的错误，并且会引起错误，这将引起hash表中100%的冲突，由于hash表中使用hash方法返回的值作为关键字。

你也应当实现一个类型相等的检查方法，依照下面的格式`isEqualTo<#class-name-without-prefix#>`:如果可能，避免调用调性检查直接调用类型相等方法是更好的选择。

一个完整的isEual*方法的模式应当如下：

```
- (BOOL)isEqual:(id)object {
    if (self == object) {
      return YES;
    }

    if (![object isKindOfClass:[ZOCPerson class]]) {
      return NO;
    }

    return [self isEqualToPerson:(ZOCPerson *)object];
}

- (BOOL)isEqualToPerson:(Person *)person {
    if (!person) {
        return NO;
    }

    BOOL namesMatch = (!self.name && !person.name) ||
                       [self.name isEqualToString:person.name];
    BOOL birthdaysMatch = (!self.birthday && !person.birthday) ||
                           [self.birthday isEqualToDate:person.birthday];

  return haveEqualNames && haveEqualBirthdays;
}
```

给定对象实例，`hash`的计算应当是确定的，这在对象被加进容器中（`NSArray`,`NSSet`,`NSDictionary`）时是尤其重要的，否则行为是没法定义的（所有容器对象使用对象hash来查找，并且强迫包含特定的属性如对象特征）。这就是说，hash的计算总是由不变的属性来计算或者更好的是保证对象的不变性。

#分类

我们知道这是非常丑陋的，但是分类应当以你自己的小写前缀以及一个下划线为前缀，例如`- (id)zoc_myCategoryMethod`。 这个实践也是[苹果推荐](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html#//apple_ref/doc/uid/TP40011210-CH6-SW4)的。

这样做是绝对必要的，因为实现一个存在于其他扩展对象或分类中的名字的方法，会导致不确定的行为。实践来说，最后一个家在的分类的方法是被调用的方法。

如果你想确定你没有用你的分类替换已有的实现，你可以设置环境变量`OBJC_PRINT_REPLACED_METHODS`为`YES`，这样做将在控制台打印出被替换的方法的名字。写代码时，LLVM5.1 不会对此提示任何警告或者错误，所有仔细一点不要覆盖了分类中的方法。

好的实践是分类名称前面使用前缀。

例如：

```
@interface NSDate (ZOCTimeExtensions)
- (NSString *)zoc_timeAgoShort;
@end
```

不要写成

```
@interface NSDate (ZOCTimeExtensions)
- (NSString *)timeAgoShort;
@end
```

分类可以用来在头文件中将相关的方法分组。这在苹果的框架中是常见的实践。（最近被提议的是`NSdate`头文件的抽取）。我们强烈鼓励在你的代码中也这么做。

在我们的实践中，创建分组对于将来重构有好处：在类或者声明开始增长时，这是一个信号，你的类做了太多的事情，因此违背了单一职责原则，之前创建的分组可以用来更好的理解不同的责任，并且在更多的自包含组件下分拆类有所帮助。

```
@interface NSDate : NSObject <NSCopying, NSSecureCoding>

@property (readonly) NSTimeInterval timeIntervalSinceReferenceDate;

@end

@interface NSDate (NSDateCreation)

+ (instancetype)date;
+ (instancetype)dateWithTimeIntervalSinceNow:(NSTimeInterval)secs;
+ (instancetype)dateWithTimeIntervalSinceReferenceDate:(NSTimeInterval)ti;
+ (instancetype)dateWithTimeIntervalSince1970:(NSTimeInterval)secs;
+ (instancetype)dateWithTimeInterval:(NSTimeInterval)secsToBeAdded sinceDate:(NSDate *)date;
// ...
@end
```

#协议

objective-c中最大的缺失是最近十年出来的抽象接口相关的东西。术语“接口"典型的是指类的`.h`文件，对应的，对于java程序员来说，它有一个广为人知的含义，即主要的用来描述一组不依赖具体实现的方法。

在objective-c中后一种情况用协议来实现。由于历史原因，通常而言协议（java中使用的接口）没有在objective-c的代码中被社区广泛的应用，主要的原因是大量的苹果开发的代码没有拥抱这个趋势，几乎所有的开发者都倾向于跟随苹果的模式以及指导方针。苹果几乎仅仅在委托模式时使用协议。抽象接口的概念是非常有力的，根植于计算机科学历史，没有理由假装它在objecive-c中使用不了。

接下来将通过具体的例子展示一个非常有效的协议的使用：从一个非常坏的设置结构开始，一直到达一个非常好并且可充用的代码片段。

展示的示例是一个RSS 订阅器的实现（想想这是一个技术面试中被问通用的测试任务）。

要求非常直接，在table view中展示一个远程的RSS订阅器。

一个非常简单的实现是创建一个`UITableViewController`的子类，并且写全部的用来检索订阅数据的逻辑代码，解析与展示在同一个地方，换句话说，一个MVC（臃肿的View Controller）。这能够工作，但是是个非常坏的设计，并且非常不幸，这在一些不那么要求的的科技创业的面试中通过。

微小的进步是遵循单一职责原则，并且穿件最少两个组块来做不同的任务：

* 一个订阅分析起来分析从一个源收集的数据
* 一个订阅读取器来展示结果

这些类的借口可能像这样：

```
@interface ZOCFeedParser : NSObject

@property (nonatomic, weak) id <ZOCFeedParserDelegate> delegate;
@property (nonatomic, strong) NSURL *url;

- (id)initWithURL:(NSURL *)url;

- (BOOL)start;
- (void)stop;

@end
```

```
@interface ZOCTableViewController : UITableViewController

- (instancetype)initWithFeedParser:(ZOCFeedParser *)feedParser;

@end
```

`ZOCFeeedParser`用一个指向一个源的`NSURL`初始化来拉取RSS订阅（详细的可能使用NSXMLParse和NSXMLParseDelegate创建有意义的数据），并且`ZOCTableViewController`用一个解析器来初始化。 我们想要它展示解析器检索出来的结果，我们用下面协议的委托来做这件事：

```
@protocol ZOCFeedParserDelegate <NSObject>
@optional
- (void)feedParserDidStart:(ZOCFeedParser *)parser;
- (void)feedParser:(ZOCFeedParser *)parser didParseFeedInfo:(ZOCFeedInfoDTO *)info;
- (void)feedParser:(ZOCFeedParser *)parser didParseFeedItem:(ZOCFeedItemDTO *)item;
- (void)feedParserDidFinish:(ZOCFeedParser *)parser;
- (void)feedParser:(ZOCFeedParser *)parser didFailWithError:(NSError *)error;
@end
```

我要说这是一个完美的合理合适的处理RSS的协议。view controller在公共借口里遵循该协议。

```
@interface ZOCTableViewController : UITableViewController <ZOCFeedParserDelegate>
```

最终的创建的代码像这样：

```
NSURL *feedURL = [NSURL URLWithString:@"http://bbc.co.uk/feed.rss"];

ZOCFeedParser *feedParser = [[ZOCFeedParser alloc] initWithURL:feedURL];

ZOCTableViewController *tableViewController = [[ZOCTableViewController alloc] initWithFeedParser:feedParser];
feedParser.delegate = tableViewController;
```

到目前为止一切很好，你可能对这个新代码很高兴，但是这些代码到底有多少能被有效的重用呢？ view controllers仅仅能够处理类型是`ZOCFeedParser`的对象，从这点上说，我们仅仅将代码分成了两部分，除了责任划分之外没有任何额外的，实在的价值。

view controller的责任应当是“展示提供的条目”，但是如果我们允许仅`ZOCFeedParser`传入,这是不可能完成的。这儿面临一个view controller中使用一个更加通用类型的对象。

我们引入`ZOCFeedParserProtocol`来修改我们的订阅分析器（在ZOCFeedParserProtocol.h 文件中，`ZOCFeedParserDelegate`依然在那）

```
@protocol ZOCFeedParserProtocol <NSObject>

@property (nonatomic, weak) id <ZOCFeedParserDelegate> delegate;
@property (nonatomic, strong) NSURL *url;

- (BOOL)start;
- (void)stop;

@end

@protocol ZOCFeedParserDelegate <NSObject>
@optional
- (void)feedParserDidStart:(id<ZOCFeedParserProtocol>)parser;
- (void)feedParser:(id<ZOCFeedParserProtocol>)parser didParseFeedInfo:(ZOCFeedInfoDTO *)info;
- (void)feedParser:(id<ZOCFeedParserProtocol>)parser didParseFeedItem:(ZOCFeedItemDTO *)item;
- (void)feedParserDidFinish:(id<ZOCFeedParserProtocol>)parser;
- (void)feedParser:(id<ZOCFeedParserProtocol>)parser didFailWithError:(NSError *)error;
@end
```

注意委托协议现在处理遵循我们新协议的对象，并且`ZOCFeedParser`接口文件更加简洁。

```
@interface ZOCFeedParser : NSObject <ZOCFeedParserProtocol>

- (id)initWithURL:(NSURL *)url;

@end
```

由于`ZOCFeedParser`现在遵循`ZOCFeedParserProtocol`,他必须实现所有被要求的方法。

从这点上说，view controller 能接受任何遵循这个新协议的对象，确定该对象将响应`start`和`stop`方法并且它将通过委托属性提供信息。这是所有的viewcontroller 应当知道的关于所给对象的信息，并且没有实现细节应当关心。

```
@interface ZOCTableViewController : UITableViewController <ZOCFeedParserDelegate>

- (instancetype)initWithFeedParser:(id<ZOCFeedParserProtocol>)feedParser;

@end
```

上面代码片段的修改看起来可能非常小儿科，但是实际上这是一个巨大的提升，因为viewcontroller将依赖约定工作，而不是具体的实现。这可以有很多好处：

* viewcontroller 能够接受任何通过委托属性提供信息的对象：这可能是一个RSS 远程订阅分析器，或者甚至是从本地数据库拉取数据的服务
* 订阅分析器对象能够完全的服用（就像第一步重构之后）
* `ZOCFeedParser`和`ZOCFeedParseDelegate`能够被其他的模块复用
* `ZOCViewController`（UI 逻辑除外）能被复用
* 由于能够使用一个假的遵循期望协议的对象，所以很容易测试

当你实现一个协议时，你应当努力坚持[替换原则](http://en.wikipedia.org/wiki/Liskov_substitution_principle)。该原则列出你应当在不破坏客户端或者实现的基础上能够用接口（objective-c中的协议）的一个实现替换另外一个。


换句话说，这意味着，你的协议不应当泄露实现类的细节；设计协议表达的抽象时要格外的小心，并且时刻记得，背后的实现是不相关的，真正重要的是抽象暴露给用户的抽象。

所有被设计将来重用的代码都是更好的代码，虽然有些现在没有展示出来，并且应当一直是程序员的目标。如此被设计的代码是经验丰富的程序员与新程序员的不同之处。

这块提及的最终代码可以在[这](http://github.com/albertodebortoli/ADBFeedReader)找到。

#NSNotification

当你定义你自己的`NSNotification`时，你应当将通知的名字定义为常量字符串。就像任何常量字符串，应当在共有接口中声明为extern，在相应的私有实现中定义相应的变量。常量的值应当是反过来的DNS标示符。

```
// Foo.h
extern NSString * const ZOCFooDidBarNotification

// Foo.m
NSString * const ZOCFooDidBarNotification = @"com.zenobjective-c.ZOCFooDidBarNotification";
```

#美化代码

#空格

* 用4个空格缩进。永远不要用tab来缩进。确保在xcode偏好中设置
* 方法或者其他（`if/else/switch/while`等）花括号开头和该语句在同一行，在新的一行结束

优选的

```
if (user.isHappy) {
    //Do something
}
else {
    //Do something else
}
```

不推荐的

```
if (user.isHappy)
{
  //Do something
} else {
  //Do something else
}
```

* 在方法之间恰好有一个空行，使得视觉上更加清晰，更加有组织。方法中的空白应当分割功能，但是通常情况下应当用一个新的方法
* 优先选择自动synthesis，但是如果必要，`@synthesize`和`@dynamic`在实现中应当是每行声明。
* 应当总是使用冒号对齐的方法调用。有一些情况，一个方法签名可能含多于3个冒号，冒号对齐使得代码更有可读性。总是用冒号对其方法，即使包含block。

优选的

```
[UIView animateWithDuration:1.0
                 animations:^{
                     // something
                 }
                 completion:^(BOOL finished) {
                     // something
                 }];
```

不推荐的

```
[UIView animateWithDuration:1.0 animations:^{
    // something 
} completion:^(BOOL finished) {
    // something
}];
```

如果自动缩进使得代码可读性变差，那么在之前将block声明为变量或者重新考虑你的方法签名。

#换行

既然这个风格指南是为了打印以及在线阅读，所以换行是非常中重要的。

例如

```
self.productsRequest = [[SKProductsRequest alloc] initWithProductIdentifiers:productIdentifiers];
```

像上面的一长行代码应当被转到第二行，参照这个风格指南空格章节（2个空格）


```
self.productsRequest = [[SKProductsRequest alloc] 
  initWithProductIdentifiers:productIdentifiers];
```

#括号

对于如下的使用 [Egyptian括号风格](https://en.wikipedia.org/wiki/Indent_style#K.26R_style)

* 控制结构（if-else，for， swith）

如下的使用非Egyptian括号风格：

* 类的实现（如果有的话）
* 方法的实现

#代码组织

引用Matt Thompson 一句话

> 代码组织是卫生问题

我们非常认同这句话。代码组织简洁以及形成的习惯，是一种展示对将来读并且修改你的代码的你自己以及其他人的尊重的方式（你自己的将来考虑在内）

#利用代码块

一个非常模糊的GCC行为，Clang也是支持的。这就是用代码块返回最后一条语句值的能力，该代码块要求用圆括号闭合。

```
NSURL *url = ({
    NSString *urlString = [NSString stringWithFormat:@"%@/%@", baseURLString, endpoint];
    [NSURL URLWithString:urlString];
});
```

这个功能可以很好的组织来分组一些块代码，这些代码通常必须是仅仅用来设置一个类。这可以带给代码阅读者一个非常重要的视觉线索以及帮助减少噪音以便专注函数中最重要变量。额外的，这个技术有一个优势，就是所有的变量在代码块内部声明，如同期待的一样，仅仅在该作用域内部有效，这意味着你没有污染整个方法的堆栈，并且你能重用一个变量名而不至于重复命名。

#编译指示

##pragma mark

`#pragma mark -` 是类中组织代码的一个很好的方式，帮助你对方法的实现分组。

我们建议使用`#pragma mark -`来划分：

* 函数功能分组的函数
* 协议实现
* 重写父类的方法

```
- (void)dealloc { /* ... */ }
- (instancetype)init { /* ... */ }

#pragma mark - View Lifecycle

- (void)viewDidLoad { /* ... */ }
- (void)viewWillAppear:(BOOL)animated { /* ... */ }
- (void)didReceiveMemoryWarning { /* ... */ }

#pragma mark - Custom Accessors

- (void)setCustomProperty:(id)value { /* ... */ }
- (id)customProperty { /* ... */ }

#pragma mark - IBActions

- (IBAction)submitData:(id)sender { /* ... */ }

#pragma mark - Public

- (void)publicMethod { /* ... */ }

#pragma mark - Private

- (void)zoc_privateMethod { /* ... */ }

#pragma mark - UITableViewDataSource

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath { /* ... */ }

#pragma mark - ZOCSuperclass

// ... overridden methods from ZOCSuperclass

#pragma mark - NSObject

- (NSString *)description { /* ... */ }

```


上面的标记将帮助视觉的区分以及组织代码。之所以这么做的一个原因是你可以cmd加点击标记调到符号定义。

要清醒的认识到，即使编译指示是一个匠人情怀程序员的签名，但是这不是一个使得你的类中方法数目无限增长的一个好的原因：拥有太多的方法应当是你的类有太多职责的一个警告标示，以及一个好的重构机会。

#关于编译指示的注解

在[http://raptureinvenice.com/pragmas-arent-just-for-marks](http://raptureinvenice.com/pragmas-arent-just-for-marks)有关于编译指示的很好讨论，这块摘录一部分。

虽然大部分iOS开发者不接触很多编译选项，但有些选项确实对控制怎么样严格检查你的代码中的错误非常有用。有时，即你想在你的代码中使用一个编译指示直接产生一个例外，这个目的可以通过临时关掉编译器的某些行为来做到。

当你使用ARC时，编译器为你插入内存管理调用。 有些情况，即编译器会陷入困惑。这样的一个例子是当你使用`NSSelectorFromString`时，有一个动态的名字的selector被调用。因此ARC不知道该方法是什么，应当使用什么样的内存管理，你讲得到`performSelector may cause a leak because its selector is unknown`的警告。

如果你知道你的代码不会产生内存泄露，你可以对这个实例去除警告，需要像这样包裹代码：

```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"

[myObj performSelector:mySelector withObject:name];

#pragma clang diagnostic pop
```

注意我们怎样通过在代码周围压入以及弹出改变来使-Warc-performSelector-leaks检查失效。这保证我们不会使检查全局失效，那样是非常严重的错误。

全部单可以生效或者失效的选项可以在[clang用户手册](http://clang.llvm.org/docs/UsersManual.html)中发现来学习关于他们的一切。

禁止对未使用变量的警告。告诉一个你定义的变量未被使用是非常有用的。大多数情况下，你想删除这些对象指向来提高效率（不管多轻微），但是有些时候你想保留他们。为什么？因为他们可能将来会有用处或者函数功能被临时删掉了。不管怎么样，一个更加聪明的禁止该警告的方式是使用`#pragma unused()`,而不是很粗暴的在相应的行注释

```
- (void)giveMeFive
{
    NSString *foo;
    #pragma unused (foo)

    return 5;
}
```

现在你可以在没有编译器警告的情况下将代码保留。并且的确，编译指示应当这些令人不快代码的下面。

#显式的警告以及错误

编译器是一个机器：它使用一组定义在clang中的规则来标记处你代码中什么是错误的。但是，通常你比编译器更加聪明。通常，你可能发下了一些讨厌的代码，你知道这些代码将导致问题，但是由于某些原因，你不能立刻自己修复它。你可以显示的发出错误，像下面这样：

```
- (NSInteger)divide:(NSInteger)dividend by:(NSInteger)divisor
{
    #error Whoa, buddy, you need to check for zero here!
    return (dividend / divisor);
}
```

相似的你可以发出警告

```
- (float)divide:(float)dividend by:(float)divisor
{
    #warning Dude, don't compare floating point numbers like this!
    if (divisor != 0.0) {
        return (dividend / divisor);
    }
    else {
        return NAN;
    }
}
```

#文档字符串

所有的主要的方法，接口，分类以及协议声明应当附有相应的描述目的以及在更大的场景中的如何作用的注释。更多的示例可以看google风格指南 大体[文件和声明注释](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml#File_Comments)一块。

总结来说，有两种风格的文档字符串，长格式以及短格式。

短格式的文档字符串完全在一行，包括注释斜线。它被用在简单函数，尤其对于那些不是公开API一部分的（可是不意味着专门这么用）

```
*// Return a user-readable form of a Frobnozz, html-escaped.*

```


注意文字被指定为一个动作（return）而不是描述（returns）。

如果描述超过了一行，你应当转到长格式文档字符串：一个概要行（物理上的）,前面一个开放的块注释，有两个/**该行的两个星，以句号，问号或者惊叹号，后面一个空行，再后面是其他的文档字符，从与第一个语句第一行相同的光标位置开始，以单独一行注释块的结束块结束。

```
/**
 This comment serves to demonstrate the format of a docstring.

 Note that the summary line is always at most one line long, and
 after the opening block comment, and each line of text is preceded
 by a single space.
*/
```

一个行数除非符合下面所有的条件，否则应当由文档字符串

* 外部不可见
* 非常短
* 含义非常明显

文档字符串应当描述函数调用的语法以及语义，而不是实现。

#注释

当需要的时候，注释应当被用来解释一块代码用来干什么。所有使用的注释应当保持最新的或者直接删除。

整块的注释应当尽量避免，同时代码应当尽可能的自解释，仅需要间歇性，很好几行的注释。例外：这不是用用来产生文档的注释。

#头文档

可能的话，类的文档应当使用Doxygen/AppleDoc语法在.h文件中书写。方法以及属性应当有文档。

例如

```
/**
 *  Designated initializer.
 *
 *  @param  store  The store for CRUD operations.
 *  @param  searchService The search service used to query the store.
 *
 *  @return A ZOCCRUDOperationsStore object.
 */
- (instancetype)initWithOperationsStore:(id<ZOCGenericStoreProtocol>)store
                          searchService:(id<ZOCGenericSearchServiceProtocol>)searchService;
```


#内部对象通信

每个重要的软件都是在需要与其他对象通信的许多对象基础上建立以完成复杂目标的。这章是关于一些设计考虑，关于第一次深入解释军火库怎么样实现很棒的结构以及是怎么工作的的。

##blocks

Block是多年来其他语言中众所周知的被称为匿名函数(lambdas)或者闭包（closures）的objective-c版本。

它们是非常好的设计一部API的方式，例如

```
- (void)downloadObjectsAtPath:(NSString *)path
                   completion:(void(^)(NSArray *objects, NSError *error))completion;

```

设计像上面这样的一些东西时，尝试声明带不止一个block的函数或者方法，并且总是把blocks最为最后的参数。尝试将data与error放在一个block而不是放在两个分开的block（通常一个成功block一个失败block）中是一份好的方法、

你应当这么做，原因如下：

* 通常有部分代码是两者之间共享的（例如：解除进度条或者活动指示器）
* 苹果就是这么做的，这样做是于平台保持一致的好方法。
* 由于block典型的是多行代码，如果block不是最有一个参数就会破坏调用点。
* 使用多余一个block座位参数使得调用点在长度长很可能比较笨拙。也会增加复杂性。

考虑上面的方法，completion block的签名非常通用：第一个参数含义为调用者感兴趣的数据，第二个参数是碰到的错误。因此，约定应当如下：

* 如果`object`不是nil，那么`error`一定是nil
* 如果`object`是nil， 那么`error`一定不是nil

由于该方法的调用者首先对实际数据感兴趣，推荐这样实现：

```
- (void)downloadObjectsAtPath:(NSString *)path
                   completion:(void(^)(NSArray *objects, NSError *error))completion {
    if (objects) {
        // do something with the data
    }
    else {
        // some error occurred, 'error' variable should not be nil by contract
    }
}
```

此外，对于异步方法，一些苹果的API 在成功的状态下向error写一些垃圾值，所以检查error可能导致误报。

###高级选项

一些关键点：

* block在栈上创建
* block被拷贝到堆上
* block有他们自己的栈值的常量拷贝（和指针）
* 可变的栈值以及指针必须用__block关键字声明

如果block没有被保留在其他地方，将保留在栈中并且当栈结构返回时将释放。当在栈上时，block对于它使用的东西的存储以及生命周期没有影响。如果block需要在栈退出后依然存在，它们可以被拷贝到堆上，这是一个显式的操作。这种方式下，block将或得引用计数，就如果cocoa中所有其他对象。当它们被拷贝后，它们带上了被它们捕获的作用，保留任何它们引用的对象。如果一个block引用一个栈上的值或者指针，那么当block被初始化时，它将被赋予那个值或者指针它自己的拷贝，该拷贝为不可变的，所以赋值是没有效果的。当一个block被拷贝，block引用的`__block`栈值被拷贝到堆上，而且在拷贝操作之后，栈上的block与堆上新的block都引用堆上的值。

LLDB 展示了block是一篇非常漂亮的事情。

![block debug](/images/blocks_debugger.png)

要注意的最重要的是`__block`值和指针在block里面被当做结构体对待，而且很明显，拥有实际的值/对象的引用。

在objective-c运行时中block是一等公民：它们有一个`isa`指针，该指针定义一个类，并且通过该类objective-c运行时可以获取方法已经存储。在非ARC的环境中，你讲毫无以为的陷入困境，难以捉摸的指针引起崩溃。`__block`仅被用在在block中使用的变量时，对于lock简单的说；

> 嗨，这个指针或者原始的类型挂靠在栈中的地址。请用一个新的栈中值引用这个小朋友。我的意思是... 两方解除引用的使用，不要保留改对象。谢谢先生。

有些时候，在声明之后，block调用之前对象已经被释放并销毁，block的执行将引起崩溃。`__block`值北邮在block里面保留。深入的讲，都是关于指针，引用，解除引用以及引用计数等。

####self上的引用循环

在使用block以及异步分发时不要陷入引用循环是非常重要的。总是对任何值设置`weak`引用可以解决引用循环。此外，将指向block的属性设置为nil是一个好的实践，这可以打破潜在的由该block捕获作用域引入的引用循环。

例如：

```
__weak __typeof(self) weakSelf = self;
[self executeBlock:^(NSData *data, NSError *error) {
    [weakSelf doSomethingWithData:data];
}];
```

不要

```
[self executeBlock:^(NSData *data, NSError *error) {
  [self doSomethingWithData:data];
}];
```

多条语句的示例：

```
__weak __typeof(self)weakSelf = self;
[self executeBlock:^(NSData *data, NSError *error) {
  __strong __typeof(weakSelf)strongSelf = weakSelf;
  [strongSelf doSomethingWithData:data];
  [strongSelf doSomethingWithData:data];
}];
```

不要

```
__weak __typeof(self)weakSelf = self;
[self executeBlock:^(NSData *data, NSError *error) {
  [weakSelf doSomethingWithData:data];
  [weakSelf doSomethingWithData:data];
}];
```

你应当将这两行加入到xcode的代码片段中，并且像这样一的准确的使用他们

```
__weak __typeof(self)weakSelf = self;
__strong __typeof(weakSelf)strongSelf = weakSelf;
```

这里是我们更加深入的关于细致事情的探究，即关于`__weak`和`__strong`限定符在block中self应用的考虑。总结来说，我们在block中有三种方式引用self:

1. 直接在block中使用`self`关键字
2. 在block外面声明`__weak`引用self，在block里面通过这个weak引用使用该对象
3. 在block外面声明`__weak`引用self，在block里面创建`__strong`引用weak引用

*情况1： 在block中使用self关键字*

如果我们直接在block中使用self关键字，在block声明时该block内部对象就被保留（实际上当block被拷贝，但是为了简化的缘故，我们暂时抛开它）。一个不变的到self的引用存在block内部，这影响该对象的引用计数。如果block被其他类或者传递给其他我们想让保留self的对象，就像其他block中使用的对象一样，因为执行block需要他们。

```
dispatch_block_t completionBlock = ^{
    NSLog(@"%@", self);
}

MyViewController *myController = [[MyViewController alloc] init...];
[self presentViewController:myController
                   animated:YES
                 completion:completionHandler];
```

没有什么大问题，但是。。 如果block被self的一个属性引用（例如下面的例子），因此对象（self）引用block？

```
self.completionHandler = ^{
    NSLog(@"%@", self);
}

MyViewController *myController = [[MyViewController alloc] init...];
[self presentViewController:myController
                   animated:YES
                 completion:self.completionHandler];
```

这就是广为人知的引用循环，并且引用循环通常应当被避免。我们从clang收到的警告如下：

```
Capturing <span class="err">'self<span class="err">' strongly in this block is likely to lead to a retain cycle
```

接下来就是`__weak`限定词

*情况2： 在block外面声明__weak引用，并在block里面使用*

在block外面声明`__weak`self 引用，在block里面通过该weak引用访问该对象可以避免引用循环。这就是如果block已经被self的一个属性所引用时我们通常想做的。

```
__weak typeof(self) weakSelf = self;
self.completionHandler = ^{
    NSLog(@"%@", weakSelf);
};

MyViewController *myController = [[MyViewController alloc] init...];
[self presentViewController:myController
                   animated:YES
                 completion:self.completionHandler];
```

在这个示例中，block没有保留对象，对象在一个属性中保留了block。酷。我们确定我们可以安全的访问self了，最坏情况，它被外面某些人置为nil。问题是，在block的作用域内self被销毁（deallocated）可能怎样？

考虑一个block被作为一个属性从一个对象拷到另外一个对象（假设再说myConroller）。之前的对象在被拷贝对象有机会执行前然后被释放。

下一步非常有趣

*情况3：在block外面声明__weak的self引用，在block内部使用__strong引用*

你可能回想这是一个在block内部使用self而不引起引用循环的小技巧。其实并非如此。对self的强引用在block运行时被创建，相比来说，在block中使用self在block声明时就评估好了，这就是保留对象。

[苹果文档](http://developer.apple.com/library/mac/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)说“对于非平凡的循环，你应当使用如下方法”:

```
MyViewController *myController = [[MyViewController alloc] init...];
// ...
MyViewController * __weak weakMyController = myController;
myController.completionHandler =  ^(NSInteger result) {
    MyViewController *strongMyController = weakMyController;
    if (strongMyController) {
        // ...
        [strongMyController dismissViewControllerAnimated:YES completion:nil];
        // ...
    }
    else {
        // Probably nothing...
    }
};

```

起初，这个示例我看起来似乎是错的。如果block被`completionHandler`属性保留， self怎么可能在外部被销毁并置为nil呢。`completionHandler`属性可以被声明为`assign`或者`unsafe_unretained`，以便允许在block被传递后对象被销毁。

我不能察觉这样做的原因。如果其他对象需要该对象(self),被传递的block将保留该对象，因此block不应当被赋给一个属性。没有`__weak/__strong`应当在这里使用。

不管怎样，在其他情况下，weakSelf是可能变为nil的，就是第2种情况展示的一样（在block外部声明weak引用，在内部使用该引用）

此外，苹果的不重要的block指什么？不重要的block是没有被传递的block，被用在定义的非常好，控制的非常好的作用域内部，因此weak限定符的使用仅仅用来避免引用循环。

就像许多[参考](https://github.com/AFNetworking/AFNetworking/issues/807)，书([有效的objective-c2.0](http://www.effectiveobjectivec.com/),[iOS和OSX高级多线程与内存管理](http://www.amazon.it/Pro-Multithreading-Memory-Management-Ios/dp/1430241160))中讨论的这个边界条件，这个话题没有被大量的开发者理解。

在block中使用强引用的真正收获是对于抢占优先的鲁棒性。再次看一下上面三种情况，在block的执行过程中。

*情况1：在block中使用self关键字*

如果block被一个属性保留，那么就创建了一个self与block之间的引用循环，两个对象都再也不能销毁了。如果block传递给其他对象并拷贝，self每次copy是都被保留一次。

*情况2：在block外面使用__weak引用，在里面使用该引用*

无论block是否被属性保留，都不会存在引用循环。如果block被传递给其他对象并被拷贝，当执行是，weakself可能变为nil。

block的执行可能会被抢占，不同的weakself指针的计算顺序可能导致不同的结果（例如，weakself在确定计算中可能变为nil）

```
__weak typeof(self) weakSelf = self;
dispatch_block_t block =  ^{
    [weakSelf doSomething]; // weakSelf != nil
    // preemption, weakSelf turned nil
    [weakSelf doSomethingElse]; // weakSelf == nil
};
```

*情况3：block外部声明一个__weak引用，在block内部使用__strong引用*

无论block是否被一个属性保留也没有引用循环。如果block被传递，拷贝给其他对象，当执行时，weakself可能已经变为nil了。当强引用被复制，并且不是nil，我们可以确定该对象在整个block的执行过程中就算抢占发生都被保留，因此stongself的序列执行将会始终如一，并且会得到相同的结果，这是由于该对象被保留了。通常情况如果strongSelf计算为nil，执行将返回，由于block可能不能够执行。

```
__weak typeof(self) weakSelf = self;
myObj.myBlock =  ^{
    __strong typeof(self) strongSelf = weakSelf;
    if (strongSelf) {
      [strongSelf doSomething]; // strongSelf != nil
      // preemption, strongSelf still not nil
      [strongSelf doSomethingElse]; // strongSelf != nil
    }
    else {
        // Probably nothing...
        return;
    }
};
```

在ARC的环境中，如果尝试使用`->`符号获取一个实例变量，编译器自己会提示错误。错误非常清晰：

```
Dereferencing a __weak pointer is not allowed due to possible null value caused by race condition, assign it to a strong variable first.
```

可以用下面代码展示：

```
__weak typeof(self) weakSelf = self;
myObj.myBlock =  ^{
    id localVal = weakSelf->someIVar;
};
```

最后

* *情况1*： 应当仅仅用在block没有被赋值给一个属性的情况下，否则将导致一个引用循环
* *情况2*：应当用在block被赋给一个属性之时
* *情况3*：这与并发执行有关，当异步的服务被调用，传递给它们的block将在过后一段时间内被执行，不确定self对象是否存在


##委托和数据源

委托是苹果框架中广泛使用的模式，同时也是Gang of Four的书“设计模式”中最重要的模式之一。委托模式不是直接的，消息发送者（委托者）需要知道消息接受者（被委托者），除了这没有其他。对象之间的耦合松了，发送者仅仅知道被委托者遵循某个特殊协议。

纯形式来说，委托是关于提供给被委托这回调方法，这意味着被委托者实现一组返回void的方法。

不行的是，这个多年来没有被苹果的API遵守，因此开发者模仿这个引导错误的方法。经典的例子是[UITableViewDelegate](https://developer.apple.com/library/ios/documentation/uikit/reference/UITableViewDelegate_Protocol/Reference/Reference.html)协议，当有些方法是void返回类型，看起来像回调：

```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;
- (void)tableView:(UITableView *)tableView didHighlightRowAtIndexPath:(NSIndexPath *)indexPath;
```

其他没有被定义成那样：

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath;
- (BOOL)tableView:(UITableView *)tableView canPerformAction:(SEL)action forRowAtIndexPath:(NSIndexPath *)indexPath withSender:(id)sender;
```

当委托者向被委托者要一些信息是，直接的含义是从被委托者到委托者，并且不在是其他方式。这概念上是不同的，而后一个新的名字将被用来描述这个模式：数据源

有人肯呢过争论说，对数据源来说苹果有[UITableViewDataSource](https://developer.apple.com/library/ios/documentation/uikit/reference/UITableViewDataSource_Protocol/Reference/Reference.html)协议，但是实际上它被用来提供真实信息应当怎么展示的信息，

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;
```

此外在上面两个方法中，苹果混合了展示层和数据层，这是非常丑陋的，并且最后，长时间以来非常少的开发者对此觉得很烂，甚至这儿我们调用委托方法，两个方法是void返回类型，以及愚蠢的非void。

为了分离概念，应当使用下面的方法：

* 委托模式： 当委托者需要通知被委托者有事件发生
* 数据源模式；当委托者需要从数据源拉取信息。

这儿是具体的例子

```
@class ZOCSignUpViewController;

@protocol ZOCSignUpViewControllerDelegate <NSObject>
- (void)signUpViewControllerDidPressSignUpButton:(ZOCSignUpViewController *)controller;
@end

@protocol ZOCSignUpViewControllerDataSource <NSObject>
- (ZOCUserCredentials *)credentialsForSignUpViewController:(ZOCSignUpViewController *)controller;
@end

@protocol ZOCSignUpViewControllerDataSource <NSObject>

@interface ZOCSignUpViewController : UIViewController

@property (nonatomic, weak) id<ZOCSignUpViewControllerDelegate> delegate;
@property (nonatomic, weak) id<ZOCSignUpViewControllerDataSource> dataSource;

@end
```

委托方法必须总是将调用者作为第一个参数，就像上面例子中一样，否则被委托对象不能够区分不同的委托者。换句话说，如果调用者没有传给被委托者对象，任何被委托者没有办法处理两个委托者，所以下面的代码是无法容忍的：

```
- (void)calculatorDidCalculateValue:(CGFloat)value;
```

默认的，协议中的方式是要求在被委托者中实现的。标记一些为可选是可能的，显式的关于必须的方法使用`@required`和`optional`关键字，像这样：

```
@protocol ZOCSignUpViewControllerDelegate <NSObject>
@required
- (void)signUpViewController:(ZOCSignUpViewController *)controller didProvideSignUpInfo:(NSDictionary *);
@optional
- (void)signUpViewControllerDidPressSignUpButton:(ZOCSignUpViewController *)controller;
@end
```

对于可选方法，委托者在发送消息到被委托者之前必须检查被委托者是否实际上实现了特定的方法，（否则会崩溃）

```
if ([self.delegate respondsToSelector:@selector(signUpViewControllerDidPressSignUpButton:)]) {
    [self.delegate signUpViewControllerDidPressSignUpButton:self];
}
```

##继承

有时候你需要重写委托方法。考虑你有两个UIViewController子类的情况：UIViewControllerA 和UIViewControllerB，类继承如下：

```
UIViewControllerB < UIViewControllerA < UIViewController
```

`UIViewControllerA` 遵循`UITableViewDelegate`并且实现`- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath.`

你想在B类中提供不同的该方法的实现，像下面这种实现可以工作：

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    CGFloat retVal = 0;
    if ([super respondsToSelector:@selector(tableView:heightForRowAtIndexPath:)]) {
        retVal = [super tableView:self.tableView heightForRowAtIndexPath:indexPath];
    }
    return retVal + 10.0f;
}
```

但是如果给的方法在父类（`UIViewControllerA`）中没有实现会怎么样

下面的调用

```
[super respondsToSelector:@selector(tableView:heightForRowAtIndexPath:)]
```

将使用NSObject的实现查找，背后实际是在`self`的上下文中，很清楚，self实现了该方法，但是程序在下一行时会崩溃，错误如下：

``` 
Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[UIViewControllerB tableView:heightForRowAtIndexPath:]: unrecognized selector sent to instance 0x8d82820'
```

在这种情况下，我们需要问一个特定类的实例变量是否响应一个给定的选择器。下面的代码就是处理这个的小技巧

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    CGFloat retVal = 0;
    if ([[UIViewControllerA class] instancesRespondToSelector:@selector(tableView:heightForRowAtIndexPath:)]) {
        retVal = [super tableView:self.tableView heightForRowAtIndexPath:indexPath];
    }
    return retVal + 10.0f;
}
```

像上面的代码是非常丑的，通常找到一种不需要重写委托方法的架构是更好的选择。

##多重委托

多重委托是一个非常基本的概念，不幸的是，大量的开发者对此非常不熟悉，往往用NSNotifcation替代。就像你注意到的一样，委托和数据源是仅仅两个对象相关的相互通信面试：一个委托者，一个被委托者。

数据源模式被前世是1对1，由于发送者要的信息仅仅能够被一个，且只有一个对象提供。对于委托模式，情况不是这样的，有很多被委托对象在等待回调是完美的原因。

有些情况，最少有两个对象对收到特定委托者的回调感兴趣，后者想知道它所有的被委托者。 这个方法描述了一个更好的分发系统，并且更加通用大的软件中信息流是怎么样的复杂的流动。

多重委托可以通过多种方法达成，读者可以挑战找到一个适当的个人的实现。一个非常灵巧的多重委托的实现使用了Luca Bernardi在[LBDelegateMatrioka](https://github.com/lukabernardi/LBDelegateMatrioska)中给出的转发机制。

这儿给出一个基本试下来呈现相应的概念。即使在cocoa中有一些办法在数据结构中存储weak引用来避免引用循环，这里，我们使用一个类来持有指向被委托对象的引用，就像单个委托所做的一样。

```
@interface ZOCWeakObject : NSObject

@property (nonatomic, weak, readonly) id object;

+ (instancetype)weakObjectWithObject:(id)object;
- (instancetype)initWithObject:(id)object;

@end
```

```
@interface ZOCWeakObject ()
@property (nonatomic, weak) id object;
@end

@implementation ZOCWeakObject

+ (instancetype)weakObjectWithObject:(id)object {
    return [[[self class] alloc] initWithObject:object];
}

- (instancetype)initWithObject:(id)object {
    if ((self = [super init])) {
        _object = object;
    }
    return self;
}

- (BOOL)isEqual:(id)object {
    if (self == object) {
        return YES;
    }

    if (![object isKindOfClass:[object class]]) {
        return NO;
    }

    return [self isEqualToWeakObject:(ZOCWeakObject *)object];
}

- (BOOL)isEqualToWeakObject:(ZOCWeakObject *)object {
    if (!object) {
        return NO;
    }

    BOOL objectsMatch = [self.object isEqual:object.object];
    return objectsMatch;
}

- (NSUInteger)hash {
    return [self.object hash];
}

@end
```

使用weak对象来实现多重委托的一个简单组成如下

```
@protocol ZOCServiceDelegate <NSObject>
@optional
- (void)generalService:(ZOCGeneralService *)service didRetrieveEntries:(NSArray *)entries;
@end

@interface ZOCGeneralService : NSObject
- (void)registerDelegate:(id<ZOCServiceDelegate>)delegate;
- (void)deregisterDelegate:(id<ZOCServiceDelegate>)delegate;
@end

@interface ZOCGeneralService ()
@property (nonatomic, strong) NSMutableSet *delegates;
@end
```

```
@implementation ZOCGeneralService
- (void)registerDelegate:(id<ZOCServiceDelegate>)delegate {
    if ([delegate conformsToProtocol:@protocol(ZOCServiceDelegate)]) {
        [self.delegates addObject:[[ZOCWeakObject alloc] initWithObject:delegate]];
    }
}

- (void)deregisterDelegate:(id<ZOCServiceDelegate>)delegate {
    if ([delegate conformsToProtocol:@protocol(ZOCServiceDelegate)]) {
        [self.delegates removeObject:[[ZOCWeakObject alloc] initWithObject:delegate]];
    }
}

- (void)_notifyDelegates {
    ...
    for (ZOCWeakObject *object in self.delegates) {
        if (object.object) {
            if ([object.object respondsToSelector:@selector(generalService:didRetrieveEntries:)]) {
                [object.object generalService:self didRetrieveEntries:entries];
            }
        }
    }
}

@end
```

使用`registerDelegate:`和`deregisterDelegate:`方法,可以简单的连接/断开组件之间的联系：如果某时一个被委托者适时对收到委托者的回调不感兴趣了，他有一个机会取消订阅。

这在有不同的view等一些回调来更新展示的信息时非常有用：如果一个view是临时的隐藏（但是仍然活着），取消对这些回调的订阅对他来说是有意义的。


#面向切面编程

面向切面编程(AOP)是在objective-c社区中没有被很好了解的，但是本应当被了解，由于运行时是非常强大的以至于AOP应当是第一个闪现在头脑中的事情。

不幸的是，由于没有标准的事实上的库，苹果没有准备好任何事情来创造性的使用 并且这个主题离重要还很远，开发者现今仍没有关心它。

引用Wikipedia中[面向切面编程](http://en.wikipedia.org/wiki/Aspect-oriented_programming)中的说法

> 一个切面可以通过在多个被称为切入点（检测是否一个给定的加入点合适）的量化或者分析指定的加入点（程序中的点）使用通知（额外的行为）来替换基本代码（非切面部分程序）的行为，

用objective-c中的语言说，这意味着使用动态特性对特定的方法添加切面。切面带来的额外行为可能为：

* 加入在特定类的特定方法被调用之前执行的代码
* 加入在特定类的特定方法被调用之后执行的代码
* 加入替代特定类特定方法原本实现的执行代码

有很多方法来实现这个，这里我们不想深入探究，基本的，它们所有的都利用了运行时的强大功能。

[Peter Steinberger](https://twitter.com/steipete) 写了一个库，[Aspects](https://github.com/steipete/Aspects)完美符合AOP过程。我们发现它可读性非常好，也被设计的非常好，这儿我们将使用它做简单的探究。

对所有AOP库来说，这些库都用运行时做了一些很酷的魔法，替换、增加方法（相比于swizzling技术更加有技巧）。

Aspect的API非常有趣并且很强大

```
+ (id<AspectToken>)aspect_hookSelector:(SEL)selector
                      withOptions:(AspectOptions)options
                       usingBlock:(id)block
                            error:(NSError **)error;
- (id<AspectToken>)aspect_hookSelector:(SEL)selector
                      withOptions:(AspectOptions)options
                       usingBlock:(id)block
                            error:(NSError **)error;
```


例如，下面的代码将在`myClass`的`myMethod:`方法（必须是实例方法）执行之后执行block参数。

```
[MyClass aspect_hookSelector:@selector(myMethod:)
                 withOptions:AspectPositionAfter
                  usingBlock:^(id<AspectInfo> aspectInfo) {
            ...
        }
                       error:nil];
```

换句话说，block参数中提供的代码将在`myClass`类的所有对象（或者是类方法的话，就是`MyClass`自身，）的`@selector`参数被执行之后执行。


我们为`MyClass`类的`myMethod:`方法添加了一个切面：

通常来说，AOP用来实现横切关系。利用这个完美的例子是分析以及日志。

接下来，我们将展示使用AOP来进行分析。分析是iOS工程中非常流行的功能，有大量的选择，从google analytics，Flurry， MixPanel等。它们大多数都有怎样跟在每个类中添加几行代码来踪特定的view以及事件的教程。

在 Ray Wenderlich的博客中，有一篇在你的view controller中包含一些示例代码来完成用[google ananlytics](https://developers.google.com/analytics/devguides/collection/ios/)跟踪事件的[长文](http://www.raywenderlich.com/53459/google-analytics-ios)

```
- (void)logButtonPress:(UIButton *)button {
    id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];
    [tracker send:[[GAIDictionaryBuilder createEventWithCategory:@"UX"
                                                          action:@"touch"
                                                           label:[button.titleLabel text]
                                                           value:nil] build]];
}
```
上面的代码在一个button被按下之后发送一个包含上下文的事件。当你想跟踪屏幕视图时情况变得更糟：

```
- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];

    id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];
    [tracker set:kGAIScreenName value:@"Stopwatch"];
    [tracker send:[[GAIDictionaryBuilder createAppView] build]];
}
```

这对于一个经验丰富的工程师应当视为代码腐朽的气息。实际上我们添加几行本不属于那儿的代码使得view controller很脏乱，这是由于跟踪事件不是view controller的职责。 你可能争论说，你有一个特定的用来负责分析跟踪的对象，并且你讲该对象注入到view controller，但是问题依旧在那，无论你将跟踪逻辑藏在哪： 你最终都会归结到在`viewDidAppear`方法中插入几行代码。

我们可以使用AOP在指定的`viewDidAppear`方法中追踪屏幕视图，并且，更进一步，我们可以使用同样的方法在其他我们感兴趣的方法中添加事件追踪，例如当用户按下一个按钮(例如：琐碎的相应IBAction的调用)

这种方法是非常干净并且不容易察觉的

* view controller将不被那些自然不属于它们的代码污染
* 对所以添加到我们代码中的切面指定一个SPOC（单点配置）文件成为可能
* SPC应当在程序开启时就被用来添加切面
* 如果SPOC文件是有缺陷的，那么最少有一个选择器或者类没有被发现，那么程序在开启时会崩溃（这对于我们的目的来说很酷）
* 公司中负责分析的团队通常提供一个需要追踪的列表文档，这些文档然后很容易转化为SPOC文件
* 由于追踪的逻辑现在是抽象的，宽展成更多分析提供者成为可能
* 对屏幕视图来说，在SPOC文件中指定相关类（相应的切面将被加入`viewDidAppear:`）就够了对于事件，指定选择器是必须的。为了发送屏幕视图和事件，一个跟踪标签以及可能附加元数据被需要用来提供额外的信息（依赖分析提供者）

你可能想要一个类似于下面的SPOC文件（.plist文件也是非常合适的）

```
NSDictionary *analyticsConfiguration()
{
    return @{
        @"trackedScreens" : @[
            @{
                @"class" : @"ZOCMainViewController",
                @"label" : @"Main screen"
                }
             ],
        @"trackedEvents" : @[
            @{
                @"class" : @"ZOCMainViewController",
                @"selector" : @"loginViewFetchedUserInfo:user:",
                @"label" : @"Login with Facebook"
                },
            @{
                @"class" : @"ZOCMainViewController",
                @"selector" : @"loginViewShowingLoggedOutUser:",
                @"label" : @"Logout with Facebook"
                },
            @{
                @"class" : @"ZOCMainViewController",
                @"selector" : @"loginView:handleError:",
                @"label" : @"Login error with Facebook"
                },
            @{
                @"class" : @"ZOCMainViewController",
                @"selector" : @"shareButtonPressed:",
                @"label" : @"Share button"
                }
             ]
    };
}
```

这块提到的结构在github上[EF Education First](https://github.com/ef-ctx/JohnnyEnglish/blob/master/CTXUserActivityTrackingManager.m)

```
- (void)setupWithConfiguration:(NSDictionary *)configuration
{
    // screen views tracking
    for (NSDictionary *trackedScreen in configuration[@"trackedScreens"]) {
        Class clazz = NSClassFromString(trackedScreen[@"class"]);

        [clazz aspect_hookSelector:@selector(viewDidAppear:)
                       withOptions:AspectPositionAfter
                        usingBlock:^(id<AspectInfo> aspectInfo) {
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
                NSString *viewName = trackedScreen[@"label"];
                [tracker trackScreenHitWithName:viewName];
            });
        }];

    }

    // events tracking
    for (NSDictionary *trackedEvents in configuration[@"trackedEvents"]) {
        Class clazz = NSClassFromString(trackedEvents[@"class"]);
        SEL selektor = NSSelectorFromString(trackedEvents[@"selector"]);

        [clazz aspect_hookSelector:selektor
                       withOptions:AspectPositionAfter
                        usingBlock:^(id<AspectInfo> aspectInfo) {
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
                UserActivityButtonPressedEvent *buttonPressEvent = [UserActivityButtonPressedEvent eventWithLabel:trackedEvents[@"label"]];
                [tracker trackEvent:buttonPressEvent];
            });
        }];

    }
}
```

#参考

这里是一些调到风格指南的苹果文档

* [objective-c编程语言](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html)
* [cocoa基础指南](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html)
* [cocoa编码指南](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
* [iOS App编程指南](http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)
* [苹果objective-c惯例](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)

其他的

* [objective-c整洁之道](http://objclean.com/)写出一个xcode集成下写objective-c代码的标准
* [Uncrustify](http://uncrustify.sourceforge.net/)源码美化

#其他objective-c风格指南

这里有一些提到风格指南的苹果文档，如果本文中有没有提及的，那么很有可可能在下面这些详情里

* [objective-c编程语言](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html)
* [cocoa基础指南](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html)
* [cocoa编码指南](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
* [iOS App编程指南](http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)

来自社区

* [NSTimes objective-c风格指南](https://github.com/NYTimes/objetive-c-style-guide)
* [google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml)
* [github](https://github.com/github/objective-c-conventions)
* [Adium](https://trac.adium.im/wiki/CodingStyle)
* [Sam soffes](https://gist.github.com/soffes/812796)
* [CocoaDevCentral](http://cocoadevcentral.com/articles/000082.php)
* [Luke Redpath](http://lukeredpath.co.uk/blog/my-objective-c-style-guide.html)
* [Marcus Zarra](http://www.cimgf.com/zds-code-style-guide/)
* [Ray Wenderlich](https://github.com/raywenderlich/objective-c-style-guide)
