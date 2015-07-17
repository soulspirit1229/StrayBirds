---
layout: post
title: Tips
category: Tips
comments: true
---


# Tips-never use rescue exception
在Ruby中，Exception是Ruby exception hierarchy的顶点。所以当你使用rescue exception的时候，你已经rescue了（全世界的）所有Exception, 包括SyntaxError,LoadError,Interrupt.

我们来进入看看这几个错误。
当你rescue Interrupt， 那么你就阻止了用户使用ctrl＋c来退出程序。
当你rescue SignalException， 那么你就阻止程序正确的对signals做出反应。它只能被kill -9 杀死，其他的都不能让它退出进程。
当你使用SyntaxError，那么eval就会悄无声息的失败。

你可以使用这段程序来执行一下反应

~~~rb
loop do
  begin
    sleep 1
    eval "djsakru3924r9eiuorwju3498 += 5u84fior8u8t4ruyf8ihiure"
  rescue Exception
    puts "I refuse to fail or be stopped!"
  end
end
~~~

所以，使用StandardError和比StandardError更精确的错误来代替Error。

~~~rb
begin
  #
rescue => e
  #
end
~~~
