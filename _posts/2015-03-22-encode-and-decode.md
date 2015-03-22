---
layout: post
title: 编码与解码
category: articles
---

现在大部分网站以及平台都采用了`UTF-8`编码，大部分情况不用考虑内容编码的问题，对于字符串编码的关注也就渐渐淡忘，甚至变的模糊起来，新接触的同学可能连字符编码的概念都没有。今天就来梳理一下字符编码。

# 编码是什么？

在计算机的世界里所有的内容都是二进制流，二进制流与概念的映射就是编码。最简单的编码就是烽火台，点火为1，标示敌人入侵，0标示和平。在复杂点的如11代表3，111代表7。在进一步用一段二进制流表示一张图片。在这些中，怎么样解释二进制流就是编码。

# 编码与解码

假设我们有A，B两套规则，能将两个不同的二进制流a, b 都映射为概念k，那么当我们从输入（计算机里就是文件）拿到a串，经过规则A得到K，但是这时候我们需要将K概念输出为b，因为有一套只知道B规则的系统需要处理K概念，那么就需要将K经过规则B转换为b交给目标系统。这样我们就完成了一个转码过程。从二进制流到概念为decode，从概念到二进制流为encode。

# 编解码错误

假设现在有一段二进制流a，但是却被错误的用B规则进行了解释，结果得到概念是L，这就是出现了编解码错误。


# python中的编解码

python内部使用的是unicode编解码规则，但是输入输出可以是GBK，UTF-8，UTF16等编码规则。python中unicode替代了上面提到的概念的作用。现在我们看一下python中出现几种情况

### 硬编码

python内部使用unicode，但是python脚本文件不一定使用，如果在python脚本中不是使用`u'**'`而是用`‘**’`来表示字符串，那么就会出现硬编码，也就是该字符串与该脚本（代码字符串）的编码格式相同。这样会出现一旦对该脚本进行转码，脚本内部依赖于编码的逻辑就会出问题。比如有一个逻辑是将某个字符串按照UTF-8转为unicode，然后在进行处理，该脚本在为UTF-8时没有问题，如果变为GBK问题就来了，就会出现将GBK码的字符串按照UTF-8进行转码的错误。


### IDE与输入输出

IDE与输入输出也只是支持某一种编码，也就是说只有某一种编码的二进制流才能正确显示为概念。这样就会出现在debug时，某个与IDE，输入输出支持编码不相符的编码字符串不能正确显示，因为该字符串被按照IDE支持的编码规则解释了。

以IDE处理UTF-8编码为例，unicode字符串会自动转为UTF-8显示，非UTF-8以及unicode的字符串就会显示不正常。分情况如下

* unicode 字符串显示不正常：这种情况下需要考虑是哪种编码decode到该unicode的，可能选择的编码错了。处理办法就是将unicode字符串按照之前错误的编码encode成二进制流。然后尝试对该二进制流按照新的编码进行decode得到正确的概念。
* 非unicode，UTF-8显示不正常： 这种情况只需要对该二进制流按照正确的编码进行decode为unicode就好。如果需要保存为UTF-8，在encode为UTF-8即可。

需要注意的又两点：

1. 搞清楚正在处理的字符串到底是什么编码
2. 现在处理的系统能接受什么编码


# 示例（scrapy抓取编码错误修正）

比如我们用scrapy抓取http://law.npc.gov.cn/treecode/home.cbs网页内容，由于http://law.npc.gov.cn/treecode/home.cbs为GB2312编码，所以我们在

```python
	def parse(self, response):
		sel = Selector(response)
		anchors = sel.xpath('//a')
		for anchor in anchors:
			text = anchor.xpath('text()').extract()[0]
```

得到的response内容的body应当是GB2312编码，但是scrapy对此进行了错误的解码，而是以CP1252编码进行解码得到unicode, 也就是text为错误解码的unicode字符串，显示就是错误的。修复该问题有多种方法

* 用 `chardet.detect(response.body)`得到`response.body`编码，然后在传给Selector之前用replace方法生成新的response， 即

```python
content_type = chardet.detect(response.body)
response = response.replace(encoding = content_type['encoding'])
```

* 对text进行转换，我们知道text是对GB2312编码字符串进行了错误解码产生的unicode字符串。那么我们先按照错误编码进行encode，然后正确解码既可以

```python
text = text.encode(reponse.encoding)
content_type = chardet.detect(text)
text = text.decode(content_type['encoding'])
```

上面两种方法上一种在整个Selector处理之前，对打段字符串进行detect，更安全。但是如果我们确认原来编码为GB2312，那么就不用detect，直接写GB2312即可，得到的text就是正确的unicode字符串。




[old link](http://tomorrow-also-bad.blog.163.com/blog/static/203002244201302683435496)

