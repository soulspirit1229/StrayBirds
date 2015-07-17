---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Reading Rails - ActiveSupport Module

## alias method chain

~~~rb
class Person
  def greeting
    "Hello"
  end
end

module GreetingWithExcitement
  def self.included(base)
    base.class_eval do
      alias_method_chain :greeting, :excitement
    end
  end

  def greeting_with_excitement
    "#{greeting_without_excitement}!!!!!"
  end
end

Person.send(:include, GreetingWithExcitement)

p = Person.new
p.greeting ###"Hello!!!!!"
~~~
使用alias_method_chain a, b
这样会生成两个方法，a_with_b， a_without_b.

我们来看源代码

~~~rb
 def alias_method_chain(target, feature)
    ActiveSupport::Deprecation.warn("alias_method_chain is deprecated. Please, use Module#prepend instead. From module, you can access the original method using super.")

    # Strip out punctuation on predicates, bang or writer methods since
    # e.g. target?_without_feature is not a valid method name.
    aliased_target, punctuation = target.to_s.sub(/([?!=])$/, ''), $1
    yield(aliased_target, punctuation) if block_given?

    with_method = "#{aliased_target}_with_#{feature}#{punctuation}"
    without_method = "#{aliased_target}_without_#{feature}#{punctuation}"

    alias_method without_method, target
    alias_method target, with_method

    case
    when public_method_defined?(without_method)
      public target
    when protected_method_defined?(without_method)
      protected target
    when private_method_defined?(without_method)
      private target
    end
  end
~~~
#### alias method
这个方法的主要功能是使用了alias\_method。
alias\_method without\_method,target
alias\_method target, with\_method

先用without_method来替换原方法，然后将原方法单名字换成with_method。这样就可以使用with_method

#### public_method_defined public
public target： 将target这个方法的权限改为public。

## alias attribute

~~~rb
  def alias_attribute(new_name, old_name)
    module_eval <<-STR, __FILE__, __LINE__ + 1
      def #{new_name}; self.#{old_name}; end          # def subject; self.title; end
      def #{new_name}?; self.#{old_name}?; end        # def subject?; self.title?; end
      def #{new_name}=(v); self.#{old_name} = v; end  # def subject=(v); self.title = v; end
    STR
  end
~~~

####module_eval
往类中增加方法new_name,new_name?,new_name=,内部实现用old_name实现。

## anonymous

## attr internal reader / attr internal writer

~~~rb
	self.attr_internal_naming_format = '@_%s'

    def attr_internal_define(attr_name, type)
      internal_name = attr_internal_ivar_name(attr_name).sub(/\A@/, '')
      # use native attr_* methods as they are faster on some Ruby implementations
      send("attr_#{type}", internal_name)
      attr_name, internal_name = "#{attr_name}=", "#{internal_name}=" if type == :writer
      alias_method attr_name, internal_name
      remove_method internal_name
    end
~~~

首先attr_internal_ivar_name的作用是format，比如我们传人的attr_name是：goro，那么
"@_%s" % :goro的结果就是@_goro.

然后给_goro添加attr_reader或者attr_writer的方法。然后alias_method，用:goro来代替_goro
然后remove_method	 internal_name.
这个internal_name是为了给有些ruby解释器，可以让代码执行的更快。

## mattr_accesor

~~~rb
  def mattr_reader(*syms)
    options = syms.extract_options!
    syms.each do |sym|
      raise NameError.new("invalid attribute name: #{sym}") unless sym =~ /^[_A-Za-z]\w*$/
      class_eval(<<-EOS, __FILE__, __LINE__ + 1)
        @@#{sym} = nil unless defined? @@#{sym}

        def self.#{sym}
          @@#{sym}
        end
      EOS

      unless options[:instance_reader] == false || options[:instance_accessor] == false
        class_eval(<<-EOS, __FILE__, __LINE__ + 1)
          def #{sym}
            @@#{sym}
          end
        EOS
      end
      class_variable_set("@@#{sym}", yield) if block_given?
    end
  end
  alias :cattr_reader :mattr_reader
~~~

class_eval(<<-EOS,__FILE__,__LINE__+1)当程序出错的时候，会指示是哪个文件，哪行报的错。
cattr_reader跟mattr_reader是同一个意思。

## concern

~~~rb
class Module
  module Concerning
    # Define a new concern and mix it in.
    def concerning(topic, &block)
      include concern(topic, &block)
    end
    def concern(topic, &module_definition)
      const_set topic, Module.new {
        extend ::ActiveSupport::Concern
        module_eval(&module_definition)
      }
    end
  end
  include Concerning
end
~~~
我们看到Module这个class中直接include一个Concerning。
