---
layout: post
title: Rack源码解析
category: articles
---

[Rack](https://github.com/rack/rack)是ruby的一个web服务统一抽象接口，该接口统一了web服务器与ruby应用程序的接口规范问题。可以对比CGI是语言无关的web服务器与应用程序的接口（输入是环境变量与标准输入，输出是标准输出）来理解Rack。 该接口定义了一个`call`函数

```ruby
def call(env)
	[code, headers,body]
end
```

其中`env`，`headers`为hash对象，`body`为响应each方法的对象。实现该方法的都可作为实现Rack接口web服务器的应用程序。实现Rack接口的对象也称为中间件(middleware)。

## Rack接口一些基本实现

Rack中包含了一些实现了`call`函数的部件，包括URLMap，Builder等，这些部件对rack对象进行组装或者实现特定的功能。

##### URLMap

  URLMap包含了一个key为URL，value为Rack接口对象的hash，它的初始化方法参数就是该hashmap。其rack接口的实现为
  
 ```
 def call(env)
 	   hashmap.each do |URL, app|
 	   		unless match(URL,env); next
 	   		// do something with env
 	   		return app.call(env)
 	   end
 	   [// error response array]
 end
  ```
  
就是在hashmap中根据url，找到合适的响应rack对象，返回该对象的`call`函数返回值。
  	
##### Builder

builder 包含了一个URLMap，在该URLmap外，增加了middleware与warmup。middleware与warmup都会接受URLMap或者里层的middleware作为参数，这两个对象在实现的时候可以根据需要决定是否调用里面的Rack对象。builder自身对 `call`函数的实现为
  
 ```
  def call(env)
  	app = wrap(URLMap,@middleware, @warmup)
  	app.call(env)
  end
 ```
  
相比于URLMap是将Rack对象串起来，Builder更像是一层层包起来。其中run函数如下

```ruby
  def run(app)
    @run = app
  end
```

如果有其他map操作，`@run`会放在URLMap最上面，path为`/`，下面`default_app`即为`@run`

```ruby
   def generate_map(default_app, mapping)
      mapped = default_app ? {'/' => default_app} : {}
      mapping.each { |r,b| mapped[r] = self.class.new(default_app, &b).to_app }
      URLMap.new(mapped)
    end
```

需要提到的是rackup命令是创建一个builder对象，然后将config.ru的代码在该对象的context下执行，参见[ruby-on-rack-2-the-builder](http://m.onkey.org/ruby-on-rack-2-the-builder)。

--------

Builder与URLMap所有的Rack对象都可以是任何一种Rack对象。这在builder的map方法中有所体现，map方法引进的是一个执行run 函数的block，该block是在新构造的一个builder的context下执行，run函数会设置该builder的`@run`值，也就是会构造出一个仅含有`@run`的builder，然后将这些builder与已有的`@run`对象组成URLMap。（该block种可能不会调用run函数，这时候`@run`对象指向上面的已有`@run`）

##### CommonLogger

common logger是一个builder提到的中间件，所以它的`call`函数实现

```ruby
	def call(env)
		code,headers,body = @app.call(env)
		log code, headers, body
		[code,header,body]
	end
```
  
  commonlogger是很多中间件实现的一个展现。这种中间件自己不产生返回值，只是对输入以及里层的Rack对象（`@app`）的输出做一些事情。
  

  
## Rack中的handler

rack中的handler就是可以调用Rack接口对象的web服务器。Rack(1.6.0)中自带的实现Rack接口的web服务器（包括一些对于其他接口的封装）包括

```ruby
    autoload :CGI, "rack/handler/cgi"
    autoload :FastCGI, "rack/handler/fastcgi"
    autoload :Mongrel, "rack/handler/mongrel"
    autoload :EventedMongrel, "rack/handler/evented_mongrel"
    autoload :SwiftipliedMongrel, "rack/handler/swiftiplied_mongrel"
    autoload :WEBrick, "rack/handler/webrick"
    autoload :LSWS, "rack/handler/lsws"
    autoload :SCGI, "rack/handler/scgi"
    autoload :Thin, "rack/handler/thin"
```
对应于源代码中的handler。上面的CGI，FastCGI，SCGI是CGI接口与Rack的封装，Mongrel，WEBrick是单独的可以运行的web服务器，不过看样子已经不再更新。其他的如[passenger](https://www.phusionpassenger.com/documentation/Design%20and%20Architecture.html)，是现在用的最为广泛的Rack框架服务器，passenger还实现了其他比如负载均衡等，可以参考上面链接。

来看一个rails -s 启动web服务时所用的自带的`Mongrel`ruby接口源码

```ruby
def self.run(app, option)
	server = Mongrel::HttpServer.new(config)
	server.register(Rack::Handler::Mongrel.new(app))
	server.run
end

def process(request,response)
	env = createEnv(request)
	setup(env)
	code,headers,body = @app.call(env)
	response.update(code,headers,body)
	// Do cleaning
end
```

启动服务器时如下

```
Rack::Handler::Mongrel.run builder
```
其中builder是最外层的rack接口，一个builder对象（由config.ru中生成, rackweb程序的启动过程就是进入ruby环境，找到config.ru文件，生成根rack对象，然后调用call方法）。其他几个webserver的rack调用接口基本类似，都是构造env，然后根据响应rack对象的返回，update某些数据。


  

