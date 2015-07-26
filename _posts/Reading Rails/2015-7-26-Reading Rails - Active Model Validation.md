---
layout: post
title: Reading Rails - Active Model Validation
category: Reading Rails
comments: true
---

# Reading Rails - Active Model Validation

validates_presence_of是从哪个类引入的。


## include,extend同一个类

首先看一个比较有意思的用法，就是同时include，extend同一个类。

~~~rb
  extend  HelperMethods
  include HelperMethods
~~~

而其实HelperMethods，是一个private method。
也就是说当我们include ActiveModel::Valliation，我们的类里有private instance methods里含有_merge_attributes这个方法，同时又有private methods含有这个方法。


~~~rb
module ActiveModel
  module Validations
    module HelperMethods # :nodoc:
      private
        def _merge_attributes(attr_names)
          options = attr_names.extract_options!.symbolize_keys
          attr_names.flatten!
          options[:attributes] = attr_names
          options
        end
    end
  end
end

~~~
当然不仅仅一个地方是HelperMethods,其实Validations下面的字文件夹中的每一个类都有一个HelperMethods的定义。

我们看AbsenceValidator，它其中就具有validator的方法。

~~~rb
module HelperMethods
  def validates_absence_of(*attr_names)
    validates_with AbsenceValidator, _merge_attributes(attr_names)
  end
end
~~~
所以这些方法都会被包括进去。

## require

在ActiveModel::Validations最下面有一行，引入文件。

~~~rb
Dir[File.dirname(__FILE__) + "/validations/*.rb"].each { |file| require file }
~~~


## validates_with
在with.rb中，定义了validates_with

在上面的例子中，可以看到当我们在自己的model中使用validates_absence_of其实调用的是valiadates_with.

~~~rb
      def validates_with(*args, &block)
        options = args.extract_options!
        options[:class] = self

        args.each do |klass|
          validator = klass.new(options, &block)

          if validator.respond_to?(:attributes) && !validator.attributes.empty?
            validator.attributes.each do |attribute|
              _validators[attribute.to_sym] << validator
            end
          else
            _validators[nil] << validator
          end

          validate(validator, options)
        end
      end
~~~

我们看到validates_with会给每个attributes创建validator。
最后调用validate方法。validate方法在validations.rb中

~~~rb
      def validate(*args, &block)
        options = args.extract_options!

        if args.all? { |arg| arg.is_a?(Symbol) }
          options.each_key do |k|
            unless VALID_OPTIONS_FOR_VALIDATE.include?(k)
              raise ArgumentError.new("Unknown key: #{k.inspect}. Valid keys are: #{VALID_OPTIONS_FOR_VALIDATE.map(&:inspect).join(', ')}. Perhaps you meant to call `validates` instead of `validate`?")
            end
          end
        end

        if options.key?(:on)
          options = options.dup
          options[:if] = Array(options[:if])
          options[:if].unshift ->(o) {
            Array(options[:on]).include?(o.validation_context)
          }
        end

        args << options
        set_callback(:validate, *args, &block)
      end
~~~
主要调用的还是ActiveSupport中的set_callback方法。
