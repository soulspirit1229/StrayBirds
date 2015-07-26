---
layout: post
title: Reading Ruby - Class Methods
category: Reading Ruby
comments: true
---


# Reading Ruby - Class Methods

## class attribute, class variable,class instance variable

在Rails的源代码中，经常可以看到使用class\_attribute. 那么class\_attribute跟class_variable到底有什么区别呢？
这里有三个概念：class instance variable,class varaibles, class_attribute

### class instance variable

我们首先来看一个小问题，如何获取@tt的值

~~~rb
class M
	@tt = 'tt in M'
end
~~~

我们知道如果定义是@@tt，那么是class_variables。但是如果定义称为@tt,那么其实是class instance variable.那么我们可以怎么获取它呢？

~~~rb
irb(main):070:0> M.class_eval {@tt}
=> "tt in M"
irb(main):071:0> M.instance_eval {@tt}
=> "tt in M"
~~~

class instance variable的子类的值是跟父类不一样的。

~~~rb
class M
  @tt = 'tt in m'
end

class N < M
  @tt = 'tt in n'
end

irb(main):080:0* M.class_eval {@tt}
=> "tt in m"
irb(main):081:0> N.class_eval  {@tt}
=> "tt in n"
~~~

### class variables
跟class instance variable 不一样， class varaibles是共用一份的。

~~~rb
class M
  @@cv = 'cv in m'
end

class N < M
end

irb(main):086:0> p N.class_variable_get :@@cv
"cv in n"
=> "cv in n"
irb(main):087:0> N.class_variable_set :@@cv,  'cv in n'
=> "cv in n"
irb(main):088:0> p M.class_variable_get :@@cv
"cv in n"
=> "cv in n"
~~~

### class attribute
class attribute是rails在class的扩展方法。Subclasses can change their own value and it will not impact parent class.这点跟class instance variable很像。

~~~rb
class Base
  class_attribute :setting
end

class Subclass < Base
end

Base.setting = true
Subclass.setting            # => true
Subclass.setting = false
Subclass.setting            # => false
Base.setting                # => true
~~~

在添加class_attribute后，这个类具有三个方法。

Base.setting
Base.setting =
Base.setting?
