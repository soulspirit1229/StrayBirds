---
layout: post
title: One Day One Question
category: ODOQ
comments: true
---


# ODOQ - inside into module
当我们掌握一个module时，我们可以获取到哪些信息，夸张一点说，接下来我们要思考怎么将一个module进行解剖。

~~~rb
module Thing
  def greet
    puts "Hello!"
  end
end
~~~

## module的信息
module中存在的methods, instance_variables, class_variables,我们该怎么获取他们呢

### method

1. methods
2. method_defined?
3. method_removed(method_name)
4. method_undefined
5. instance_methods(true/false)
6. instance_method(:method_name)
7. alias_method
8. define_method
9. public\_class_methods
10. private\_class_methods
11. remove_method
12. undef_method

### instance_variables
主要是继承自Object的方法

1. instance\_varaible_defined?
2. instance_variable_get
3. instance_variable_set
4. instance_variables
5. remove_instance_variable

如果是在Rails中，还有一些功能

1. instance_values
2. instance_variable_names

### class_variables

1. class\_variable_defined?
2. class\_variable_get
3. class\_variable_set
4. class_variables
5. remove_class_variables

这里有点奇怪的是remove_instance_variable是Object的方法，而remove_class_variables是Module的方法。


### const

1. constants(true/false)
2. const_defined?
3. const_get
4. const_set
5. const_missing
6. remove_const

### 重要的

#### class_eval, class_exec
class_eval accpets a string or a block
class_exec accept a block and allow you to pass parameters to it.

~~~rb
class Array
  p self                     # prints "Array"
  43.instance_eval{ p self } # prints "43"
end

def example(&block)
  42.instance_exec("Hello", &block)
end
example{|mess| p mess, self } # Prints "Hello" then "42"
~~~

#### extended

#### included


### 其他

1. ancestors
2. autoload?(name)

我们可以看到当我们定义一个module后，其中的所有东西对我们都是可见的，可更改的，这是相当自由的，但同时我们需要妥善使用其中的一些方法，以免发生一些未知的错误。
