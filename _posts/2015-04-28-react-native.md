---
layout: post
title: react-native 
category: articles
---

一直以来，开发者在为两件事情进行着各种各样的尝试

1. 各平台公用一套代码
2. 方便的将更新的代码效果展现给用户

第一点是为了减少人力支出，统一管理业务流程，这就要求有各个平台必须要有一个平台层面的东西。
第二点则是在强运营时代所必须面对的问题，也是遇到严重线上问题是最期望能够进行的解决方案（这里不是鼓励由于放弃代码质量）。这就需要代码的动态加载执行，再更加宽泛点就是对服务端返回数据进行解释。

为了做到以上这两点，回顾一下现有的技术方案。一般来说能够动态加载的方案都会支持跨平台

* iOS与android c/c++层面的代码公用。由于iOS基于unix，android基于linux，所以这两个平台可以共有一些c/c++接口
* 语言的直接转换。如google的[j2objc][],直接在代码层面将java代码转换为objective-c代码。 [Xamarin][] 用c#来开发iOS，android，winphone，这个是C#编译生成[mach-o][]文件，java字节流。除此之外还有额外的一些语言，如[haskell可以编译为mach-o](https://wiki.haskell.org/IPhone), 最新出的swift来编写android与.net的[silver][]
* 用web页面来开发各平台需要共享的部分。各个平台都支持web浏览器，而现在移动端的web浏览器都是比较新的版本，支持h5。于此相配套的是增强web与native功能的交互，如[phonegap][]，phonegap在web中嵌入预定义好模式的url调用，web端只要调用该url就可以唤起native相应的功能。其他的还有自定制web与native交互的[WebViewJavascriptBridge][]。
* 用xml与css进行页面布局的[BeeFramework][]， BeeFramework采取了策略是将界面的展示抽取出来，替代的是storyboard，xib的功能，逻辑的实现还是需要依赖native语言。
* 脚本，如[lua][wax]在iOS平台上的实现（已经不维护了）。lua在游戏中使用较多，可以用来在线更新游戏策略。
* 最后就是今天要讨论的[react-native][]，从本质上说，react-native也是在平台上封装了一层脚本，只不过这个脚本恰好是iOS上已经支持的一些功能的javascript。


react-native是reactJS（reactJS的基本想法是将独立的展示，响应逻辑作为component进行管理）的iOS版本，实现的功能是用js+xml来实现iOS上所有功能(现在看到的是实现了大部分常用功能)，也就是对整个native功能的一整套javascript封装。所以react-native首先是可以动态加载的，大部分的业务逻辑也可以跨平台使用（除去可能与平台耦合的API的封装）。  从实现方式上看可以认为是[WebViewJavascriptBridge][]超级增强版，但是完全抛开webview的展示作用。增强的是利用自定义的数据格式，依赖objective-c runtime来进行代码执行（基于此，swift对于runtime消息动态分发的支持如果减弱，其实面临蛮大的风险）。

理解react-native如何工作，最核心的是理解javascript与native怎么进行桥接。从最基本的层面说，两种语言（javascript与objective-c）的交互一定是通过一定格式的数据进行的，比如说objective-c都能处理json数据结构，那起一个objective-c的接受json的web服务，然后根据传进来的json结构，执行预先制定好对应该json结构的objective-c逻辑代码，这里不需要关心调用该接口的语言，可以作为任意一种能输出json结构的语言与objective-c的交互。在iOS中，对于javascript不用这么麻烦，因为iOS内嵌了javascript的执行环境（javascriptcore与webview），我们只需要将js代码输入javascript执行环境，就能得到执行输出（直接为字符串输入输出，具体格式需要自己约定转换）。但是javascript 只能被动的调用，反向直接调用native代码却做不到。上面提到的phonegap，WebViewJavascriptBridge等在反向调用方面做了很多事情。react-native也必须处理反向调用。react-native具体是怎么工作的，我们通过几个问题来回答

1. javascript如何调用objective-c代码
	react-native中js对oc的相互调用与WebViewJavascriptBridge中非常相似，不同点在于WebViewJavascriptBridge通过webview的回调触发特定js代码的执行，拿到执行结果，react-native则是起了一个runloop，用CADisplayLink循环执行特定的命令得到js的执行结果，也就是oc在轮询js，而WebViewJavascriptBridge则是被动通知（js可以引发webview的回调）。相关的代码为
	
	```objc
	@interface RCTDisplayLink : NSObject <RCTInvalidating>

	- (instancetype)initWithBridge:(RCTBridge *)bridge selector:(SEL)selector NS_DESIGNATED_INITIALIZER;

	@end
	```
	
	`RCTDisplayLink`是对CADisplay的封装， 在其`init`方法中将CADisplay添加到runloop中，所以需要注意`RCTDisplayLink `的生成线程，如下所示
	
	`RCTBridge`的`setup`方法中的
	
	```
	[_javaScriptExecutor executeBlockOnJavaScriptQueue:^{
    		_displayLink = [[RCTDisplayLink alloc] initWithBridge:self 			selector:@selector(_jsThreadUpdate:)];
  	}];
	```
	
	在该方法中在js 线程中开始轮询，js线程的吊起在 Executor中
	
	```objc
	static NSThread *javaScriptThread;
  	static dispatch_once_t onceToken;
  	dispatch_once(&onceToken, ^{
   	 	// All JS is single threaded, so a serial queue is our only option.
    	javaScriptThread = [[NSThread alloc] initWithTarget:[self class] selector:@selector(runRunLoopThread) object:nil];
    	[javaScriptThread setName:@"com.facebook.React.JavaScript"];
    	[javaScriptThread setThreadPriority:[[NSThread mainThread] threadPriority]];
    	[javaScriptThread start];
  	});
	```
	`runRunLoopThread `方法会起一个runloop来处理CADisplay输入源。
	
	最后通过js执行环境获得js的执行状态以及传给oc的命令
	
	```
	- (void)_actuallyInvokeAndProcessModule:(NSString *)module method:(NSString *)method arguments:(NSArray *)args context:(NSNumber *)context
	{
	#endif
  		[[NSNotificationCenter defaultCenter] postNotificationName:RCTEnqueueNotification object:nil userInfo:nil];

  		RCTJavaScriptCallback processResponse = ^(id json, NSError *error) {
    		[[NSNotificationCenter defaultCenter] postNotificationName:RCTDequeueNotification object:nil userInfo:nil];
    		[self _handleBuffer:json context:context];
  		};

 	 [_javaScriptExecutor executeJSCall:module
                              method:method
                           arguments:args
                             context:context
                            callback:processResponse];
	}
	```
	
	
	
2. js如何知道有哪些封装好的指令oc可以执行
	
	经过1中拿到oc需要执行的命令，但是这些命令js之前是怎么知道的？最简单的方法是直接在js代码与oc都写上相互匹配的方法，但这意味着一个功能需要同时维护两份代码。需要更加简单的方式告诉js oc可以处理那些命令。这个过程就需要定义命令的格式。react-native采取了非常巧妙的方法来处理，将需要导入js执行环境的数据放在macho文件__DATA，具体是采用
	
	```
		#define RCT_IMPORT_METHOD(module, method) \
__attribute__((used, section("__DATA,RCTImport"))) \
		static const char *__rct_import_##module##_##method##__ = #module"."#method;
		
	```
	
	```
		#define RCT_EXPORT_MODULE(js_name) \
  		+ (NSString *)moduleName { __attribute__((used, section("__DATA,RCTExportModule" \
  		))) static const char *__rct_export_entry__ = { __func__ }; return @#js_name; }
	```
	
	```
		#define RCT_EXTERN_REMAP_METHOD(js_name, method) \
 		- (void)__rct_export__##method { \
    	__attribute__((used, section("__DATA,RCTExport"))) \
    	__attribute__((__aligned__(1))) \
    	static const char *__rct_export_entry__[] = { __func__, #method, #js_name }; \
  		}
	```
	三个预编译宏来标记需要注入到js执行环境的数据。对这些数据，可以通过
	
	```
	static NSArray *RCTJSMethods(void)
	static NSArray *RCTBridgeModuleClassesByModuleID(void)
	static RCTSparseArray *RCTExportedMethodsByModuleID(void)
	```
	三个方法从__DATA section中提取，提取之后在结合oc的runtime，取得相应的class，method信息，生成`RCTModuleMethod`对象，方便以后调用。最后将格式化好的数据注入到js中，等待js调用，格式化通过
	
	```
	static NSDictionary *RCTRemoteModulesConfig(NSDictionary *modulesByName)
	static NSDictionary *RCTLocalModulesConfig()
	```
	两个方法来进行， 其中`RCTRemoteModulesConfig `生成的是为js执行环境调用的，`RCTLocalModulesConfig `则是由oc调用js时所使用，区别体现在注入js的数据结构的type字段上面，一个为remote，一个为local
	
	```
	   "methodName2": {
       	"methodID": 1,
        	"type": "remote"
      	}
	```
	local标记的方法是React-Native的js模块自己定义的函数，主要的作用是将native的一些事件交给js做处理，如timer 的trigger， touch，scroll,app active等。remote则是oc提供的一些可供js调用的接口。
	

3. react-native 整体结构如何
	
	1、2中提到的是两个关键性问题，回过头来我们看一下整体结构，大致分为
	* js executor: js的执行环境，分别为javascriptCore与webview
	* bridge: oc与js的代码调用接口，1、2中的大部分功能就是bridge的功能。oc可以通过bridge来调用js代码，而js的执行环境也是bridge进行配置的。
	* modules: 2中解释的js执行环境的配置，在代码层面是通过module来管理，每一module可以用2中提到的宏标记自己，这样bridge就会将该module加载进js的执行环境，然后js就可以调取改module的功能。这里面oc中的各种视图占据了大部分，每个单独的视图功能，如UITextField，就会有对应的module RCTTextFieldManager。除去视图，其他的如App的状态，文件的操作，异常处理等都是单独的module。react-natvie为了更加模块化，还将一些module单独抽取为subproject，如图片处理，定位等。
	* css_node: react-natvie通过xml来布局，所以对css需要进行解析处理。大部分的工作在`RCTShadowView`与`RCTViewManager`进行。RCTShadowView是视图的几何属性的树状结构的节点。
	* 其他的一些工具性的代码。如`RCTKeyCommands`,`RCTLog`等
	
4. debug实现
	
	在iOS中对js进行debug比较困难。react-native的实现方式是通过websocket，用chrome浏览器执行相关js代码。具体就是上面提到的js executor不使用iOS内嵌的两种，而是使用chrome浏览器，改executor为RCTWebSocketExecutor。在该executor中，通过webscoket与浏览器执行环境连接，远程将参数发送给浏览器，浏览器执行完后在通过websocket将结果返回。这样js的debug就和普通的网页区别不大。

##ref

* [j2objc][]
* [react-native][]
* [BeeFramework][]
* [C_preprocessor][]

[C_preprocessor]: http://en.wikipedia.org/wiki/C_preprocessor
[react-native]: https://facebook.github.io/react-native/docs/getting-started.html
[j2objc]: https://github.com/google/j2objc 
[wax]: https://github.com/probablycorey/wax
[BeeFramework]: https://github.com/gavinkwoe/BeeFramework
[react-native]: https://github.com/facebook/react-native
[Xamarin]: http://www.microsoft.com/taiwan/vstudio//
[mach-o]: https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachORuntime/index.html
[WebViewJavascriptBridge]: https://github.com/marcuswestin/WebViewJavascriptBridge
[phonegap]: https://github.com/sintaxi/phonegap
[silver]: http://elementscompiler.com/elements/silver/