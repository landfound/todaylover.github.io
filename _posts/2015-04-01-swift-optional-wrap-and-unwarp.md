---
layout: post
title: swift中Optional的基本操作
category: articles
---

在objective-c中，可以将一个对象赋值为nil，对nil进行任何方法都会返回0或者nil。在使用时如果使用者忽略了对象为nil情况的逻辑，就有可能引起业务错误，甚至程序崩溃（如向数组中添加nil）。在swift中对一个对象是否可以为nil的考虑显示化，使用者必须时刻清楚使用的对象是否会有成为nil的情况，想nil值发送 这将上面提到问题更加明显的展示在使用者面前。swift使用Optional来达到这种操作。

在`swift module`中可以找到Optional(类似的`ImplicitlyUnwrappedOptional`)的定义，如下

```
enum Optional<T> : Reflectable, NilLiteralConvertible {
    case None
    case Some(T)

    /// Construct a `nil` instance.
    init()

    /// Construct a non-\ `nil` instance that stores `some`.
    init(_ some: T)

    /// If `self == nil`, returns `nil`.  Otherwise, returns `f(self!)`.
    func map<U>(f: (T) -> U) -> U?

    /// Returns a mirror that reflects `self`.
    func getMirror() -> MirrorType

    /// Create an instance initialized with `nil`.
    init(nilLiteral: ())
}

```

从上面可以看出，类型为`Optional<String>`的对象，是指该对象可以为`.None`，也可以为`.Some(String)`

```
var testNone : Optional<String>
testNone = .None // nil
testNone = .Some("DF") // some : "DF"
testNone = "ASF" // some : "ASF"
```

如果一个对象有可能被复制为nil，那么这个对象就需要为Optional<T>类型,而不是T类型。

swift对于Optional有一些简洁的语法

* `String?`对应`Optional<String>`
* `String!`对应`ImplicitlyUnwrappedOptional<String>`

swift中对Optional对象使用方法如下：

* `let value:T?`: 该方式定义了一个类型为optional<T>的对象value。 如果value不为nil，从value中抽取类型为T的值方式如下：
	1. `if let value = optionalValue {...}`,该方式首先判断optionalValue是否为nil，不为nil的话，将类型为T的值抽取出来赋值给value，并且执行条件内容。
	2. `let value = optionValue!`。这是对optionalValue进行强制抽取，如果optionalValue为nil，则程序会崩溃
	3. `let value:T? = optionValue?.somefunction()`。这是optional的链式操作。如果optionalValue为nil，则整个表达式返回nil，否则调用somefunction函数。需要注意的是这种链式表达式的返回值是optional
	4. `let value = optionvalueA ?? optionValueB`。 如果optionvalueA不为nil，则将optionvalueA进行提取，将T类型值赋给value，否则将optionValueB赋给。再次需要注意的是optionValueB可能也为Optional，这时候如果optionvalueA为nil的话，value就是optionValueB而不是B中包含的值
	5. `let value = valueA as? K` 将valueA尝试转换为类型K，如果失败，返回nil。相比于as如果转换失败就会崩溃，这块需要将value声明为Optional
* `let value:T!`: 定义一个了类型为ImplicitlyUnwrappedOptional<T>的对象value
	对于 Optional 类型所有的操作，在ImplicitlyUnwrappedOptional类型上都可已进行。额外不同的是，ImplicitlyUnwrappedOptional类型的值在需要时能自动抽取，如果抽取失败（值为nil）就会崩溃。比如可以直接写
	`let value:T = ImplicitlyUnwrappedOptionalValue`，可以直接调用`ImplicitlyUnwrappedOptionalValue.somefunction`
	
	

如果将Optional放在函数式程序的角度来看，Optional就是一个盒子，更应当关注的是在该盒子可以应用的一些函数，比如map，这些函数使得在不打开盒子面对复杂问题（该值到底是nil还是具体的值，该值所处的上下文到底是什么等）的情况下将一些操作进行下去，这些函数是通向避开复杂性或者说副作用的桥梁。


## ref

* [Swift Programming Language](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/OptionalChaining.html#//apple_ref/doc/uid/TP40014097-CH21-ID245)
* [swift optionals use](http://www.touch-code-magazine.com/swift-optionals-use-let/)
* [swift enumerations](http://ios-blog.co.uk/swift-tutorials/swift-enumerations/)
* [enumeration](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID145)
