---
layout: post
title: One Day One Question
category: ODOQ
comments: true
---


# ODOQ - 如何理解singleton_class

我们先来看一段代码

~~~rb
class Dog
  def self.about
    'we bark'
  end
end

Dog.about # a singleton method that's defined in Dog's singleton class:
Dog.singleton_class.instance_methods.include? :about # => true
~~~

Every Ruby object has a singleton class to store methods for a particular object.

~~~rb
class Object
  def my_singleton_class
    class << self
      self
    end
  end
end
~~~

返回的就是当下的singleton_class.

~~~rb
[74] pry(main)> class Fox
[74] pry(main)*   def talk
[74] pry(main)*     p 'fox talk'
[74] pry(main)*   end
[74] pry(main)* end
=> nil
[75] pry(main)>
[76] pry(main)> fox = Fox.new
=> #<Fox:0x007ff41455d678>
[77] pry(main)> Fox.singleton_class
=> #<Class:Fox>
[78] pry(main)> Fox.class
=> Class
[79] pry(main)> fox.class
=> Fox
[80] pry(main)> fox.singleton_class
=> #<Class:#<Fox:0x007ff41455d678>>
~~~
