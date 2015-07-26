---
layout: post
title: One Day One Question
category: ODOQ
comments: true
---

# ODOQ - == === eql? equal?
## ==
如果是两个Object1 == Object2，只有这两个Object是同一个Object才会返回true。

通常这个 == 方法会被overridden

## ===
三个等号的比较操作===
通常情况下这中方式与==是一样的，但是在某些特定情况下，===有特殊的含义：

1. 在Range中===用于判断等号右边的对象是否包含于等号左边的Range；
2. 正则表达式中用于判断一个字符串是否匹配模式，
3. Class定义===来判断一个对象是否为类的实例，
4. Symbol定义===来判断等号两边的符号对象是否相同。

~~~rb
(1..10) === 5 # true: 5属于range 1..10
/d+/ === "123" # true: 字符串匹配这个模式
String === "s" # true: "s" 是一个字符串类的实例
:s === "s" # true
~~~

## equal?
equal? method不应该被overridden，它一般校验内存地址是否相同

## eql?
eql? method是用来比较两个值的hash key是否相等。他是用在Hash中用来校验key的相等性。跟 == 有点区别。但在String时，又一样，主要看怎么覆盖方法的。

~~~rb
1 == 1.0     #=> true
1.eql? 1.0   #=> false
~~~
