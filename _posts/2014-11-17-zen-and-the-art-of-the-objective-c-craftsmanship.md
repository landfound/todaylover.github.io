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

```objc
if (!error) {
    return success;
}
```

不推荐的

```objc
if (!error)
    return success;
```

或者

```objc
if (!error) return success;
```

在2014年2月广为人知的[goto fail](https://gotofail.com/)在苹果的SSL/TLS实现中被发现。这个bug就是因为在if条件后一个if条件后的重复goto语句。用括号包起if分支就可以阻止这种问题发生。

代码摘录如下

```c
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

```objc
if ([myValue isEqual:@42]) { ...
```

不推荐的

```objc
if ([@42 isEqual:myValue]) { ...
```

#nil和BOOL检查

与Yoda条件相同，nil检查也在争论的核心。






















