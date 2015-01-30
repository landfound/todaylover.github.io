---
layout: post
title: 开发者工具集
category: articles
---

#iOS

### 第三方库

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
* [NYXImagesKit](https://github.com/Nyx0uf/NYXImagesKit) 图片处理的一些库，包括比例压缩，图片格式转换，图片简单滤镜

### xcode工具

* [SimulatorManager](https://github.com/tue-savvy/SimulatorManager) 快速管理simulator数据
* [VVDocumenter-Xcode](https://github.com/onevcat/VVDocumenter-Xcode ) 快速插入注释文档的xcode 插件
* [xcproj](https://github.com/0xced/xcproj) 与cocoapod配合必须的，如果不想仓库快速增加的话
* [Unused](https://github.com/jeffhodnett/Unused) 清理不必要的图片资源
* [iReSign](https://github.com/maciekish/iReSign) 重新签名的工具
* [Alcatraz](https://github.com/supermarin/Alcatraz) xcode 插件管理工具，必须安装哈
* [xctool](https://github.com/facebook/xctool) 替代xcodebuild东西，可配置化，输出友好
* [class-dump](https://github.com/nygard/class-dump) oc 查看类结构的工具 
* [chisel](https://github.com/facebook/chisel) 视图调试工具，可以debug时隐藏视图，显示视图，增加边框，以及打印出视图结构等

### 分发平台

* [fir](http://fir.im)  配合第三方企业签名修改
* [pgyer](http://pgyer.com) 自带企业发布
* [testflight](http://testflightapp.com) 被苹果收购的分发平台，网速还可以忍，已经到张家口了


### 依赖管理

* [Carthage](https://github.com/Carthage/Carthage) iOS8 引进framework后出现的framework依赖管理， 配置文件cartfile
* [CocoaPods](https://github.com/CocoaPods/CocoaPods) workspace subproject 源码管理，配置文件podfile，可以指定源码路径


### 资源

* [http://code4app.net/](http://code4app.net/) 一些iOS常用模块代码，英文
* [http://code4app.com/](http://code4app.com/) 同上，是中文的

### 高级

* [http://www.gnustep.org/](http://www.gnustep.org/) 其他GNU cocoa框架的实现，开源，可以学习学习一些基础实现
* [objccn](http://objccn.io/) obj.io 中文翻译
* [obj.io](http://obj.io) obj.io 专题iOS内容


# 综合网站

* [medium.com](https://medium.com/)
* [joelonsoftware.com](http://www.joelonsoftware.com/) 软件工程实践相关经验
* [hackernews](https://news.ycombinator.com/) 讨论质量蛮高的一个技术者网站，最新的动态
* [reddit](http://www.reddit.com/) 各种问答网站
* [IT橘子](http://itjuzi.com/) 创业者打鸡血的网站，看看都觉得这是你的时代
* [kickstarter.com](https://www.kickstarter.com/) 众筹项目
* [producthunt](http://www.producthunt.com/) 关注最新的好的产品
* [techcrunch.com](http://techcrunch.com/) 科技博客
* [designcode.io](https://designcode.io/) iOS 开发的完整引导网站
* [http://www.devstore.cn/](http://www.devstore.cn/) 开发者所需要的各种东西的集合，比如推送，短信，人脸识别
* [http://mob.com/](http://mob.com/) shareSDK，短信SDK 等

-----

* [http://luosimao.com/](http://luosimao.com/) 短信以及语音合作平台
* [http://www.yunpian.com/](http://www.yunpian.com/) 短信验证平台
* [http://open.189.cn/](http://open.189.cn/) 很贵的短信平台

-----

* [http://xamarin.com/](http://xamarin.com/) C# 跨平台开发，收费
* [http://kmspico.net/](http://kmspico.net/) windows 破解工具

-----

* [http://www.tuspark.com/](http://www.tuspark.com/)清华科技园孵化器
* [http://chkinn.com](http://chkinn.com) 创客inn，创业一站式服务
* [http://51shebao.com/](http://51shebao.com/) 代缴社保



#设计

* [uisdc.com](http://hao.uisdc.com/) 设计导航网站，好吧，其他的全在里面
* [sketch](http://bohemiancoding.com/sketch/) 简单的画矢量图的工具，[教程](https://designcode.io/sketch)
	* [sketch-measure](https://github.com/utom/sketch-measure) sketch 自动标注插件，使用sketch必须的 
	* [Sketch-Toolbox](https://github.com/buzzfeed/Sketch-Toolbox) sketch 插件管理
	* 可以在github中搜索sketch，有很多不错的工具链
* [hackdesign.org](https://hackdesign.org/) 写成程序员看的设计网站
* [http://iosfonts.com/](http://iosfonts.com/) iOS可用字体
* [http://www.zcool.com.cn/](http://www.zcool.com.cn/) 设计素材库
* [http://www.sccnn.com/](http://www.sccnn.com/) 素材库
* [http://www.pinterest.com/](http://www.pinterest.com/) pinterest图片收藏网站，可以搜索出想要的好图片，国内的有[huaban.com](http://huaban.com)
* [dribbble](https://dribbble.com/) 一些设计案例

---------------

* [tutsplus](http://tutsplus.com/) 一些设计方面的教程，有收费的也有免费的
* [unsplash](https://unsplash.com/) 每10天10张免费的高清图片，质量非常好
* [pexels](http://www.pexels.com/) 免费的高清图片
* [http://wefunction.com/8-unique-places-to-find-quality-free-photos/](http://wefunction.com/8-unique-places-to-find-quality-free-photos/) 介绍获取免费高清图片的文章
* [gratisography](http://www.gratisography.com/) 免费的图片，质量还行，
* [picjumbo](http://picjumbo.com/) 免费图片，质量感觉一般，有搜索
* [superfamous](http://superfamous.com/) 图片像素很高，但是感觉不够清亮，偏重灰暗色调，比较压抑，山石居多。
* [imcreator](http://www.imcreator.com/free)分主题，有搜索，风格不明显，资源不错

--------

* [themify](http://themify.me/) 主要是一些网页版的theme，以及一点icon
* [patterntap](http://patterntap.com/) 一些网页设计模板







#android

* [http://www.apportable.com/](http://www.apportable.com/) objective-c 接口的android
* [http://www.stellasdk.org/](http://www.stellasdk.org/) objective-c android接口




#web

###综合网站

* [bootcss.com](http://www.bootcss.com/) bootstrap的中文案例
* [getbootstrap.com](http://getbootstrap.com/) bootstrap 官网，用来快速搭建可接受的web页面
* [yui](http://yuilibrary.com/)  已经不更新了
* [YUI CN](http://www.yuicn.org/)
* [http://www.w3schools.com/](http://www.w3schools.com/) web相关基础知识大全
* [lesscss](http://lesscss.org/) css预编译语言
* [typekit](https://typekit.com/) adobe在线字体解决方案
* [http://css-tricks.com/](http://css-tricks.com/) css

### 第三方库

* [https://github.com/mcasimir/mobile-angular-ui](https://github.com/mcasimir/mobile-angular-ui) mobile web 开发
* [backbone](https://github.com/jashkenas/backbone) 
* [underscore](https://github.com/jashkenas/underscore) 一些工具性质的函数，如合并，遍历，map等
* [jiathis](http://www.jiathis.com/) 社会化分享的空间，可以快速集成
* [algolia](https://www.algolia.com) 交互式搜索

### 工具

* [jsnice](http://www.jsnice.org/) js 在线反解压


#后台

### 云服务




### 服务后台

* [https://leancloud.cn/](https://leancloud.cn/) 国内的云服务开发环境，提供对象的授权访问，可以看做云文件系统
* [https://parse.com/](https://parse.com/) 国外的云开发环境，包括push，统计等，与leancloud基本类似
* [pusher.com](https://pusher.com/) push 
* [https://pingplusplus.com/](https://pingplusplus.com/) 多方支付接口封装

### 框架

* [playframework](https://www.playframework.com) playframework , scala开发后台
* [restlet](https://github.com/restlet) MVC java web 框架
* [spring-framework](https://github.com/spring-projects/spring-framework) java MVC web 框架
* rails
	* [rails-composer](https://github.com/RailsApps/rails-composer)  rails 创建引导
	* [ruby-toolbox](https://www.ruby-toolbox.com/) 介绍ruby 常用的一些框架
	* [rails-assets](https://rails-assets.org) 自动将asset资源，javascript，css等转换为gem的源，相比于原始[gem库](https://rubygems.org/search) 的gem，工作方式是直接根据bundle，bower在线生成gem
	


### 自动部署

[capistrano](https://github.com/capistrano/capistrano) 自动化部署不同环境

### 持续集成

#协作

* [tower](https://tower.im) 轻型项目管理网站，可以简单的多人合作
* [trello](https://trello.com/) 看板网站，敏捷开发的执行
* [qq企业邮箱]()  挺好用的，基本功能免费
* [hipchat](https://www.hipchat.com) stash公司出的企业级聊天工具，集成很多第三方提醒环境的提醒，如Crashlytics崩溃，github pull request等



#代码托管

* github 付费托管
* [自己搭建git服务](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-git-server-on-a-vps)，仅仅提供git托管服务，可以配合[reviewboard](https://www.reviewboard.org/)来进行code review
* [gitlab](https://github.com/gitlabhq/gitlabhq) 搭建自己的github
* [bitbucket](http://bitbucket.org) 五个人以下的项目不收费
* [gerrit](https://code.google.com/p/gerrit/) 主要用来代码review了
* 本地git + 网盘

#理论

* [arxiv](http://arxiv.org/)数学，物理，统计学等的开放paper




