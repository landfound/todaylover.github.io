---
layout: post
title: swift,objective-c and cocoapods(0.36.0)
category: articles
---

# 混合编程

apple发布swift后出现了一大批swift写成的[第三方库](https://github.com/matteocrippa/awesome-swift)[](https://github.com/Wolg/awesome-swift.git)，虽然如此但是有很多优秀的第三方库还只有objective-c版本，如[mantle](https://github.com/Mantle/Mantle)，这就是需要swift与objective-c的混合编程，apple对于这种混合提供了支持，对于target(framework参见[apple MixandMatch][])具体来说如下

* swift 引用objective-c代码，需要将被引用的objective-c代码的header 文件添加到bridging header file中。 bridging header为普通的header头文件，但是需要在build setting 中的 swift compiler -> Objective-C Bridging Header 中进行添加，命名为*-Bridging-Header。
* objective-c 引用swift代码，则需要在引用swift文件所在module自动生成的module头文件，为modulename-Swift.h

混合编程中涉及到的很重要的一个概念就是module，module是llvm采用的用来解决include方法进行头文件引用所存在弊病的方法，具体见[llvm module][]。对于xcode来说，一个target就是一个module，在build setting中有相应的product module name字段来表示改module的名称。objective-c中引用swift添加的头文件中的modulename就为该处的modulename。对于swift来说，module是最基本的代码可见域，在一个module中，所有的swift代码都是可见的，相应的在swift中引用objective-c采用的bridging header文件就是将objective-c的声明在整个module中可见，即在该module中所有的swift代码中都可以引用在bridging header 文件中import进来的代码。运用bridging header的这个特性，可以将某个第三方module全部引入进来。swift与objective-c引用第三方module的方式分别为`import`与`@import`,其中第三方module的编写语言可以使swift与objective-c的混写。

swift与objective-c混合编程时需要注意的如下

* objective-c头文件不能引用swift自动生成的头文件，因为如果该objective-c头文件加入了该swift文件所在module的bridging header 中可能引起循环添加。
* 某些swift特有的属性不能在objective-c中使用，如模板，具体见[apple MixandMatch][]

# cocoapods

在objective-c时代，cocoapods作为依赖管理的工具非常省时省力。具体的行为就是cocoapod将第三方库编译为static libraries，然后将header文件所在位置加入system header 搜索路径，所以一般的引用方式为`#import <*>`（也可以`#import "*"`，区别参见[stackoverflow](http://stackoverflow.com/questions/1217943/where-are-include-files-stored-ubuntu-linux-gcc)）。在cocoapods 0.36.0之前，对于swift工程就需要[手动](http://badr.co.id/blog/using-objective-c-pods-in-swift-project#comment-80)将pod的header文件添加到bridging header文件中。在0.36.0之后，cocoapod引入了`use_frameworks!`，例如

```ruby
target 'wuxi' do
     use_frameworks!
     pod 'Mantle'
end
```

在podfile中加入该语句，可以强制将所有pod打成framework, 该framework 的package（区别llvm语言中的module）中defines module 为YES。在工程中就可以用import（objective-c为@import ）来引用。例如在ViewController.swift文件中可以`import Mantle;`来引入Mantle模块。

# target build过程

1. build所有pod，生成framework，具体构建与target类似
2. 使用`swift`命令编译所有swift代码，该过程会生成swiftdoc(文档)，swiftmodule（swift 中public接口以及版本、路径信息等，最终的swiftmodule由多个swiftmodule进行merge产生），modulename-Swift.h文件以及多个binary文件
3. `clang`编译objective-c代码,生成binary文件
4. 将2，3步骤生成的binary文件进行链接
5. 嵌入pod生成的framework
6. 嵌入swift运行需要的动态链接库如`libswiftCore.dylib`


## ref

* [apple MixandMatch][]
* [llvm module][]
* [pod author guide to frameworks][]
* [stackoverflow: import objective-c pod into swift project](http://stackoverflow.com/questions/27908841/old-libraries-new-cocoapods)
* [module of swift](http://andelf.github.io/blog/2014/06/19/modules-for-swift/)
* [deep module][]
* [using oc in swift project][]

[llvm module]:http://clang.llvm.org/docs/Modules.html#id11
[apple MixandMatch]:https://developer.apple.com/library/prerelease/ios/
[pod author guide to frameworks]:http://blog.cocoapods.org/Pod-Authors-Guide-to-CocoaPods-Frameworks/
[deep module]:http://railsware.com/blog/2014/06/26/creation-of-pure-swift-module/
[using oc in swift project]:http://badr.co.id/blog/using-objective-c-pods-in-swift-project
