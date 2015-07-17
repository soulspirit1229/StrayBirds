---
layout: post
title: One Day One Question
category: ODOQ
comments: true
---


# ODOQ-rescue ensure的使用
在我们写代码的时候，代码经常会抛出错误，这时没有及时的rescue或者处理错误，会出现我们不期望的结果

1. 错误直接展示给用户
2. 错误消失了，没有通知用户

那么是否有好的方法能解决这个问题呢？使用rescue,ensure就是其中的一个方法。

## 基本语法
~~~rb
	begin
	rescue SomeException => e
		#deals with exception
	rescue
		#deals with StandardError
	else
		#no exception occured
	ensure
		#something that need to be done
	end
~~~

### begin
这是最基本的语法，使用begin,rescue,ensure.
这个使用方法有一些变种，单独使用rescue,可以不写begin。

~~~rb
def do_the_work
  some_code_which_may_throw
rescue ArgumentError
  $stderr.puts "Ooops! Passed the wrong argument."
else
  puts "Job done."
end
~~~

### rescue
我们也可以使用在rescue中使用retry，那么就会不断执行。
不要使用rescue Exception => e, 如果一定要使用，需要重新raise exception

~~~rb
def method
	do_work
rescue Exception => e
	logger.error ""
	raise e
end
~~~

### else
当程序中的代码没有被抛出错误时，会执行else方法

### ensure
我们知道ruby可以在方法中返回data，而且ensure是必须执行的代码块，但是ensure的结果并不会返回，除了你显式调用return以及raise方法。

~~~rb
def my_method
	1
ensure
	return 2
end

my_method : 2
~~~
