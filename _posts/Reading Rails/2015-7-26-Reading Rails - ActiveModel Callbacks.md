---
layout: post
title: Reading Rails - Active Model Callbacks
category: Reading Rails
comments: true
---

# Reading Rails - ActiveModel Callbacks

我们先来看个例子关于如何使用ActiveModel的Callbacks的。

~~~rb
class MyModel
  extend ActiveModel::Callbacks

  define_model_callbacks :create

  def create
    run_callbacks :create do
      p 'creating ...'
    end
  end

  before_create :action_before_create

  def action_before_create
    p 'before create'
  end
end

MyModel.new.create
~~~

它相比ActiveSupport中的Callback，最主要的区别是定义了define_model_callbacks的方法。

## define\_model\_callbacks

在define\_model\_callbacks先后使用了ActiveSupport中的define_callbacks以及set_callbacks方法。

~~~rb
      callbacks.each do |callback|
        define_callbacks(callback, options)

        types.each do |type|
          send("_define_#{type}_model_callback", self, callback)
        end
      end

~~~

而\_define\_#{type}\_model\_callback中定义了set_callbacks方法，使用了singleton method.

~~~rb
    def _define_before_model_callback(klass, callback) #:nodoc:
      klass.define_singleton_method("before_#{callback}") do |*args, &block|
        set_callback(:"#{callback}", :before, *args, &block)
      end
    end
~~~

这样就可以使用run_callbacks,这跟ActiveSupport的原理是一样的。
