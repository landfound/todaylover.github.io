---
layout: post
title: devotion
category: articles
---

#iOS

## 第三方库

* [Mantle](https://github.com/Mantle/Mantle) model对象基类。基于objective-c runtime 来获取property，进行objective-c对象与其他数据结构的map操作。包括json， coredata等。也实现了NSCoding协议
* [TapkuLibrary](https://github.com/devinross/tapkulibrary) 一些基础工具集。 包块常用的消息提示控件，进度控件，日期控件，滑动高亮label控件，自定义键盘控件以及一些简单常用操作封装
* [Masonry](https://github.com/Masonry/Masonry) autolayout 自动布局的封装。 简化生成NSLayoutConstraint操作，使得布局的相对概念更加清晰的展示。
* [SDWebImage](https://github.com/rs/SDWebImage) 图片自动加载，缓存的工具。使得imageview 可以简单指定URL以及placeholder就可以加载图片。也可以用来预加载图片
* [FXBlurView](https://github.com/nicklockwood/FXBlurView) 可以很轻松做出模糊效果，支持动态模糊，原理是有view生成图片，然后对图片进行模糊。动态的话再加一个计时触发。
* [FLEX](https://github.com/Flipboard/FLEX) 强大的视图调试工具，可以动态的查看视图层级结构，附带的可以管理一些基础数据，如NSUserDefults里面的数据，是debug诡异视图问题的最佳帮手。
* [OHHTTPStubs](https://github.com/AliSoftware/OHHTTPStubs) 可以做http请求的mock，在单测，开发前期，以及边界条件的测试方面可以发挥作用
* [specta](https://github.com/specta/specta) 对苹果原生的XC的封装，使得单测的结构更加清晰，对异步测试的支持更加强大
* [CocoaSecurity](https://github.com/kelp404/CocoaSecurity) 对苹果一些底层的数据加密等操作的封装，比较方便的工具
* [TMCache](https://github.com/tumblr/TMCache) 内存，文件缓存，缓存对象生命周期边界管理还行
* [QBImagePickerController](https://github.com/questbeat/QBImagePickerController) iamgepicker 增强版，collectionview 多选
* [SWTableViewCell](https://github.com/CEWendel/SWTableViewCell)cell 侧滑操作，如苹果原生delete操作
* [JSQMessagesViewController](https://github.com/jessesquires/JSQMessagesViewController) IM 视图基本结构以及操作，配合LeanCloud或者parse做出IM 还比较容易
* [KSFileUtilities](https://github.com/karelia/KSFileUtilities) 不错的URL操作工具集，不支持Pod。。。
* [ZBar](https://github.com/ZBar/ZBar) 二维码扫描工具
* [AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit) UI流畅框架，写起来有点烦
* [AFSoundManager](https://github.com/AlvaroFranco/AFSoundManager)声音管理封装
* [FXKeychain](https://github.com/nicklockwood/FXKeychain) keychain 操作封装
* [SVPullToRefresh](https://github.com/samvermette/SVPullToRefresh) 可以满足大部分下来刷新需求
* [SVProgressHUD](https://github.com/TransitApp/SVProgressHUD) 常用界面锁死loading UI
* [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket) CFSocket简化封装，如果想象自定义协议的话，必不可少
* [GCDWebServer](https://github.com/swisspol/GCDWebServer) 好吧，iOS上可以直接搭建web服务器
* [BeeFramework](https://github.com/gavinkwoe/BeeFramework) XML 来写界面的一整套框架，整体功能很强大，用到了大量消息转发机制，很依赖约定。比较担心的是快速跟上苹果API 更新步伐问题
* [WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge) javascript 与objc的桥接，objc注册回调或者直接发送消息
* [VCTransitionsLibrary](https://github.com/ColinEberhardt/VCTransitionsLibrary) 一些navigation 的transition 效果
* [CocoaSPDY](https://github.com/twitter/CocoaSPDY) spdy 协议iOS实现，替代http协议的东西，但是推广比较慢。比较适合列表式请求
* [plcrashreporter](https://github.com/plausiblelabs/plcrashreporter) 程序崩溃后，堆栈的walker，有名的崩溃收集程序底层都是用这个。 竟然不支持short version， 自己加呗
* [TTTAttributedLabel](https://github.com/TTTAttributedLabel) 可以检测链接的label，用来将特定的字段作为链接
* [BlocksKit](https://github.com/zwaldowski/BlocksKit) 将block回调应用到各个方向的工具集。最方便的是作为observer回调使用。
* [fmdb](https://github.com/ccgus/fmdb) sqlite的操作封装
* [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 信号机制在objective-c中的工具库，基于函数式编程的解包操作bind。用来做异步数据变动的绑定操作
* [PonyDebugger](https://github.com/square/PonyDebugger) 很强到的视图调试工具，也可用来调试网络数据以及coredata。自从用了*FLEX*后就很少用了
* [AFNetworking](https://github.com/AFNetworking/AFNetworking) 网络数据请求的轻量级库，稍微自定义一下就是自己的基础库。
* [GPUImage](https://github.com/BradLarson/GPUImage) 图片处理的必备库，可以添加各种效果
* [Aspects](https://github.com/steipete/Aspects) objective-c 面向切面编程的库，依赖于objective-c runtime的方法动态替换功能

## xcode工具

* [SimulatorManager](https://github.com/tue-savvy/SimulatorManager) 快速管理simulator数据
* [VVDocumenter-Xcode](https://github.com/onevcat/VVDocumenter-Xcode ) 快速插入注释文档的xcode 插件
* [xcproj](https://github.com/0xced/xcproj) 与cocoapod配合必须凭，如果不想仓库快速增加的话
* [Unused](https://github.com/jeffhodnett/Unused) 清理不必呀的图片资源
* [iReSign](https://github.com/maciekish/iReSign) 重新签名的工具
* [Alcatraz](https://github.com/supermarin/Alcatraz) xcode 插件管理工具，必须安装哈
* [xctool](https://github.com/facebook/xctool) 替代xcodebuild东西，可配置化，输出友好
* [class-dump](https://github.com/nygard/class-dump) oc 查看类结构的工具 
* [chisel](https://github.com/facebook/chisel) 视图调试工具，可以debug时隐藏视图，显示视图，增加边框，以及打印出视图结构等



#android

#后台

## 框架

## 自动部署

## 持续集成


