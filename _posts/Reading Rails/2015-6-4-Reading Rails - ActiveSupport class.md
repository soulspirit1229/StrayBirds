---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Reading Rails - ActiveSupport class

## class_attribute

~~~rb
	   define_singleton_method(name) { nil }
      define_singleton_method("#{name}?") { !!public_send(name) } if instance_predicate

      ivar = "@#{name}"

      define_singleton_method("#{name}=") do |val|
        singleton_class.class_eval do
          remove_possible_method(name)
          define_method(name) { val }
        end

        if singleton_class?
          class_eval do
            remove_possible_method(name)
            define_method(name) do
              if instance_variable_defined? ivar
                instance_variable_get ivar
              else
                singleton_class.send name
              end
            end
          end
        end
        val
      end
~~~

#### define_singleton method

~~~rb
guy = "Bob"
guy.define_singleton_method(:hello) { "#{self}: Hello there!" }
guy.hello    #=>  "Bob: Hello there!"
~~~

#### remove_possible_method

~~~rb
  def remove_possible_method(method)
    if method_defined?(method) || private_method_defined?(method)
      undef_method(method)
    end
  end
~~~

根据代码，可以看出定义了三个方法name, name?, name=
