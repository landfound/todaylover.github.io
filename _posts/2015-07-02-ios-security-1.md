---
layout: post
title: iOS 安全
category: articles
---

由于iOS的app安装源的封闭性以及沙盒设计，iOS安全一般是指用户的数据是否安全存储，iOS客户端与服务器的通信是否安全。存储安全常见就是数据库层的加密，或者在程序中直接嵌入对称加密，相对于与服务器之间的通信来说一则攻击者不容易接触，二则一旦接触就是找到秘钥一个任务。一般秘钥放在keychain或者嵌在代码中。在这主要讨论与服务器通信的安全。一个对于通信的解决大概经历一下几个过程。

#数据抓取

最基础的数据抓取非常简单，可以用[charles][]或者[mitmproxy][]在本地起一个代理，然后将iphone的代理设置为以及设置好的代理，包括ip以及端口号。设置之后，打开要抓取的app，如果是http请求就应当能够看到相应的请求。如果是https请求，则需要在iphone上装入自签名的CA（用邮件将CA.cert发送到iphone上，然后点击安装为profile，可以随时删除），然后在代理上用该CA签发的证书作为相应的请求的https证书。这样下来就可以看到https中的内容了。如果app做了证书锁定，即只接受签名链中包含特定证书（或者公钥）的证书，这种办法就会失效。这时候需要做的就是修改源程序的锁定证书逻辑，取消证书锁定。这就是我们下面要做的事情。需要说明的一点，对于模拟器进行网络请求抓取无论http或者https都要安装CA，模拟器内部使用了https。

一旦通过代理抓到了通信的数据，那么app也就无安全可言。这里面需要说的是，https并不是那么可靠，正确的使用很关键，开启证书锁定是非常好的实践。证书锁定后，通过简单的操作抓取数据也就不可能了。

对于某些敏感的数据接口，服务器可能会进行数据校验，就是在提交给服务器的参数中有签名算法生成的签名，用意确定发出请求的数据是自己客户端发出的，并且没有经过篡改。针对此，攻击者就需要拿到签名算法，而这通常也需要对二进制代码进行
逆向,查看签名算法如何定义的。

#未加密的文件

为了了解程序的逻辑，我们需要拿到程序的执行二进制文件。在iOS中就是ipa解压后bundle中的macho可执行文件，macho的文件格式可参见[macho ABI][]。
一般该文件包含多个平台（armv7，arm64）的执行文件。可以通过

```
file "binary file path"
```

查看拿到的二进制文件的文件组成,如`binary name: Mach-O 64-bit executable x86_64`。要获得更多的关于该文件的信息可以使用苹果自带的工具`otool`,该工具会可以查看macho的格式中的特定部分。简单的说macho可以分为三部分：描述头，加载描述，数据。
其中描述头包含整个的文件组成；加载描述则是说改文件加载到内存中需要遵循的一些加载地址；数据则包含真正执行程序、符号以及数据。macho还对文件按照段与区进行了
二级描述，一个段中可以有多个区，加载描述以及数据都是按照段与区进行组织。我们可以通过`otool -h`查看描述头，`otool -l`查看加载描述，`otool -t`查看
__TEXT段下__text区的数据等等。

从appstore中直接下载的app是经过加密的，在iphone中必须经过内核解密后才能执行。所以拿到该执行文件并
不能进行直接分析，需要先解密。比较偷懒的方法是从越狱渠道获取相应app的可执行文件，越狱渠道的是没有加密的。
但是对于不进行越狱市场投放的app就不能这么做了。获取解密的执行文件基本的过程是等iphone将要执行的app的执行文件解密后，
将相应的内存拷贝出来，替换掉加密的部分。这个过程你需要一个越狱的iphone。越狱之后，在iphone上debug调起相应的程序，设置断点，
然后memory dump就可以获得未加密的二进制文件。在进行这些之前，你需要获得两个信息：

* 该二进制程序加载在内存中的什么地方。由于在iOS4.3后进行[address space layout random][],所以加载的地址是在加载的时候动态确定的。在二进制文件中有描述本应当加载在内存中地址的字段。为__PAGEZERO在记载描述字段，里面的VM offset以及VM size与 File offset，File size，共同表述了空置区从哪个偏移开始，加载在内存中的哪个地址，在文件中多长（一般为0），在内存中占多大。为了获取动态的该值可以通过lldb中产看镜像加载地址`lldb> image list`。
* 相应的二进制文件中哪些部分进行了加密，这是通过加载描述中的crypt offset与 crypt size进行描述的。需要注意的是这里的offset是指加密的数据从单个平台二进制文件开头开始的偏移值。不是指整个可能包含多个平台的二级制文件。

获取上面两个信息后，需要做的就是`lldb> memory read --binary -f 0xloadbase+0xcryptoffset  0xloadbase+0xcryptoffset+oxcryptsize`。获取解密二进制文件后需要做的就是替换未解密的文件中相对应的部分。需要注意的是计算原始二进制文件替换的起始偏移量，对于包含多平台的二进制文件要确定解密的是哪个平台的文件，然后在文件头中找到改平台的文件偏移再加上crypt offset 就是要替换的原始文件起始偏移。

对于二进制解密有比较成熟的工具，如[dumpdecrypted][],[Clutch iOS dump][],并不需要自己逐步的做。其中[dumpdecrypted][]是通过以动态链接库注入加载进行的，可以通过[blocking code injection][]提到的方法进行防止动态链接库的注入。当然用lldb进行镜像dump也会遇到一些问题，如可能遇到[防止debug][],但是lldb只要能够加载程序，就能够找到相应的保护代码，然后可以通过register write，memory write 来避过安全检查，越复杂的安全检查逻辑，如[crack prevention][]提到的在很多个地方埋检查点就会增加破解的成本。

最后需要说一下，如果觉得多平台二进制文件看的实在是太烦，可以用lipo将二进制文件按照平台分解`lipo -thin armv7 ...`

#反汇编

拿到未经加密的二进制程序后就需要对提取程序的元信息（OC由于具有动态特性，所有有很多元信息），常见的如[class-dump-z][],otool等。[class-dump-z][]可以直接获取二进制中OC类的组织结构以及类，方法名称，通过基本的阅读，可以大致了解整个代码结构。在这一层可以通过类，方法名称混淆来提高代码安全性，如[ios-class-guard][]，该工具会随机替换程序中的命名，导致攻击者很难通过名称直接获取信息，只能够通过系统调用的符号以及代码结构进行推断，那么破解的成本就会高很多。

拿到了代码的基本结构，就需要了解关心部分的具体执行逻辑，可以使用一些代码分析工具，比较常用的如[hopper][],[ida][]等，通过这些工具，通过在头文件中找到一些基本信息进行搜索，可以拿到具体的汇编代码位置，然后就可以通过这两个工具自带的反汇编工具，查看逆向的代码基本逻辑，这里就需要了解一些基本的寄存器的基本使用以及压栈退栈的内存布局，如r0~r3用来存储参数，其他参数通过压栈进行传输，r0用来存储返回值，str为存储执行，ldr为加载，mov未执行等。具体见[ios assembly tutorial][]以及[ARMv6 Function Calling Conventions][]。反汇编出来的代码有的参数解释，是不正确的，需要沿着寄存器，栈的使用轨迹上洵，修正代码逻辑。

拿到了代码逻辑后，比如说获取了在哪个地方进行了SSL pin（一般来说就是某个值设置为TRUE），那么就可以用二进制文件编辑器直接修改相应的值，然后再重新对程序包进行签名[resign iOS app][],就会拿到没有ssl绑定的程序，这个时候通过代理抓取就能拿到所有的通信数据了。对于非常复杂的一些如签名算法等，这些签名算法的函数一般都自成体系，那么可以直接将其汇编代码嵌入过来[arm asm][]，只要知道需要对那些参数进行签名就可以了。

到此，代码的静态分析基本上就结束了，对于简答的东西可以静态分析，对于比较烦的东西可以用lldb进行动态分析。

#lldb

首先第一步就是如何在越狱的iphone机器上进行调试，可以参考[DebugServer][]的描述，将xcode自带的使用的debug server进行授权签名后，通过scp传到iphone上，然后调起需要debug的程序，在某个端口监听debug的客户端。在本地用lldb起一个客户端，链接到远程debug server，就可以进行代码调试了。代码调试中最基本的就是设置断点，调试中可以通过符号以及地址设置断点，对于地址设置断点，就需要通过hopper等知道其所对应的代码逻辑。剩下的就是对函数调用堆栈的查看以当前函数的栈的查看，寄存器的查看以及修改。这个可以在[lldb 手册][]查看具体的操作。

在lldb链接过程中，有时候wifi会出现错误提示，这时候用USB做转发即可。

除了lldb进行debug也可以使用[Cycript][]来动态的执行一些OC的指令，来查看程序中的对象。Cycript是在iOS程序中起一个js与oc的桥梁，可以执行一些OC的方法，进而通过一些静态变量找到所需要查看的对象。
还有一个UI查看利器FLEX,越狱后安装可以查看View树的结构以及相关的属性，NSUserDefaults等。


做到安全也只是相对的。OC的元信息会导致很容易被逆向，所以安全相关的代码不要用OC，相关的数据也不要做成从OC的单例中可以寻踪访问到的，安全检查代码应当尽可能分散，不要模块化，更不要一个函数就搞定，很容易被一网打尽。

能够保证app与服务器的数据不被篡改就够了，但是对于那些防范请求不被伪造的程序来说就需要做更多的安全防范。另外如果静态的数据请求防伪造难达到，退一步要求统计意义上的不被伪造的就是选项之一，在这就需要理解一个合理的请求发生的场景，以及这些场景从统计意义上来说有哪些特征，提取这些特征，就能够进一步防范数据不被大规模的伪造。安全的每一步都只是开始，远未结束。

[self signed cert:]:http://stackoverflow.com/questions/10175812/how-to-create-a-self-signed-certificate-with-openssl
[tools]:http://iphonedevwiki.net/index.php/Reverse_Engineering_Tools
[Cycript]:http://iphonedevwiki.net/index.php/Cycript
[hopper]:http://www.hopperapp.com/
[ida]:https://www.hex-rays.com/index.shtml
[不能输入中文]: http://stackoverflow.com/questions/4606570/os-x-terminal-utf-8-issues
[charles]: http://www.charlesproxy.com/
[mitmproxy]: https://mitmproxy.org/
[resign iOS app]:http://dev.mlsdigital.net/posts/how-to-resign-an-ios-app-from-external-developers/
[arm asm]:http://blog.noctua-software.com/arm-asm.html
[ARMv6 Function Calling Conventions]:https://developer.apple.com/library/ios/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARMv6FunctionCallingConventions.html#//apple_ref/doc/uid/TP40009021-SW1
[ios assembly tutorial]:http://www.raywenderlich.com/37181/ios-assembly-tutorial
[pinning ssl for iOS]: http://initwithfunk.com/blog/2014/03/12/afnetworking-ssl-pinning-with-self-signed-certificates/
[decrypting iOS app]: http://timourrashed.com/decrypting-ios-app/
[Clutch iOS dump]: https://github.com/KJCracks/Clutch
[iOS dump analysis]:http://www.infointox.net/?p=123
[DebugServer]: http://iphonedevwiki.net/index.php/Debugserver#Alternative_Instructions_.2864-bit_compatible.29
[lldb 手册]: http://lldb.llvm.org/lldb-gdb.html
[dumpdecrypted]:https://github.com/stefanesser/dumpdecrypted
[blocking code injection]:http://www.samdmarshall.com/blog/blocking_code_injection_on_ios_and_os_x.html
[防止debug]: http://applidium.com/en/news/securing_ios_apps_debuggers/
[crack prevention]:http://iphonedevwiki.net/index.php/Crack_prevention
[class-dump-z]:(https://github.com/nygard/class-dump)
[macho ABI]:https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachORuntime/
[ios-class-guard]: https://github.com/Polidea/ios-class-guard
[address space layout random]:https://en.wikipedia.org/wiki/Address_space_layout_randomization
[Position Independent Executables]: https://rce64.wordpress.com/2013/01/27/private-decrypting-apps-on-ios-6-multiple-architectures-and-pie/
