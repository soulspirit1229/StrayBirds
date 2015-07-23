---
layout: post
title: One Day One Question - private method
category: ODOQ
comments: true
---

# ODOQ - private method in ruby
Here's the short and the long of it. What private means in Ruby is a method cannot be called with an explicit receivers, e.g. some_instance.private_method(value). So even though the implicit receiver is self, in your example you explicitly use self so the private methods are not accessible.

Think of it this way, would you expect to be able to call a private method using a variable that you have assigned to an instance of a class? No. Self is a variable so it has to follow the same rules. However when you just call the method inside the instance then it works as expected because you aren't explicitly declaring the receiver.

Ruby being what it is you actually can call private methods using instance_eval:

~~~rb
class Foo
  private
  def bar(value)
    puts "value = #{value}"
  end
end

f = Foo.new
begin
  f.bar("This won't work")
rescue Exception=>e
  puts "That didn't work: #{e}"
end
f.instance_eval{ bar("But this does") }
~~~

Hope that's a little more clear.

-- edit --

I'm assuming you knew this will work:

~~~
class Foo
 def public_m
  private_m # Removed self.
 end
 private
 def private_m
  puts 'Hello'
 end
end

Foo.new.public_m
~~~
