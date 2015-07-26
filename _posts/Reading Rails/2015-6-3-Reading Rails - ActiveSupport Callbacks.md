---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Reading Rails - ActiveSupport Callbacks

在ActiveSupport::Callbacks中看到一个例子，阐述了Callback的用法。

~~~rb
class Record
  include ActiveSupport::Callbacks
  define_callbacks :save

  def save
    run_callbacks :save do
      puts "- save"
    end
  end
end

class PersonRecord < Record
  set_callback :save, :before, :saving_message
  def saving_message
    puts "saving..."
  end

  set_callback :save, :after do |object|
    puts "saved"
  end
end

person = PersonRecord.new
person.save
~~~

其中主要用到了两个方法define\_callbacks以及set_callback的方法，我们主要来看下这两个方法。

## A define\_callbacks
我们先来看Callbacks里面比较重要的三个类： CallbackChain， CallbackSequence， Callback.
我们来看CallbackChain.

在define_callbacks方法中

~~~rb
      def define_callbacks(*names)
        options = names.extract_options!

        names.each do |name|
          class_attribute "_#{name}_callbacks"
          set_callbacks name, CallbackChain.new(name, options)
        end
      end

       def set_callbacks(name, callbacks)
        send "_#{name}_callbacks=", callbacks
      end
~~~
为每一个callbacks都定义了一个CallbackChain。同时定义方法_#{name}\_callbacks为CallbackChain。

接下来看CallbackChain的initialize方法，

~~~rb
class CallbackChain #:nodoc:#
      include Enumerable

      attr_reader :name, :config

      # If true, any callback returning +false+ will halt the entire callback
      # chain and display a deprecation message. If false, callback chains will
      # only be halted by calling +throw :abort+. Defaults to +true+.
      class_attribute :halt_and_display_warning_on_return_false
      self.halt_and_display_warning_on_return_false = true

      def initialize(name, config)
        @name = name
        @config = {
          scope: [:kind],
          terminator: default_terminator
        }.merge!(config)
        @chain = []
        @callbacks = nil
        @mutex = Mutex.new
      end

      def each(&block); @chain.each(&block); end
~~~

其中比较重要的是@chain,@callbacks两个成员变量。通过prepend,append,insert,delete方法可以往CallbackChain中存储变量值。

A include Enumerable

B Mutex

所以总的来说，define_callbacks就是定义_#{name}\_callbacks,接下来就是使用set_callbacks。

## B set_callbacks

~~~rb

  def set_callback(name, *filter_list, &block)
        type, filters, options = normalize_callback_params(filter_list, block)
        self_chain = get_callbacks name
        mapped = filters.map do |filter|
          Callback.build(self_chain, filter, type, options)
        end

        __update_callbacks(name) do |target, chain|
          options[:prepend] ? chain.prepend(*mapped) : chain.append(*mapped)
          target.set_callbacks name, chain
        end
      end
~~~
主要的方法之一是创建了Callback。
我们再来看看Callback.build方法

~~~rb
 class Callback #:nodoc:#
      def self.build(chain, filter, kind, options)
        new chain.name, filter, kind, options, chain.config
      end

      attr_accessor :kind, :name
      attr_reader :chain_config

      def initialize(name, filter, kind, options, chain_config)
        @chain_config  = chain_config
        @name    = name
        @kind    = kind
        @filter  = filter
        @key     = compute_identifier filter
        @if      = Array(options[:if])
        @unless  = Array(options[:unless])
      end
~~~

一个Callback的结构如下

~~~rb
=> [#<ActiveSupport::Callbacks::Callback:0x007fd56a852ea0
  @chain_config=
   {:scope=>[:kind],
    :terminator=>
     #<Proc:0x007fd566ddb100@/Users/shenghuan/.rbenv/versions/2.0.0-p598/lib/ruby/gems/2.0.0/gems/activesupport-4.1.8/lib/active_support/callbacks.rb:725 (lambda)>},
  @filter=:before_print_1,
  @if=[],
  @key=:before_print_1,
  @kind=:before,
  @name=:print,
  @unless=[]>]
~~~

set_callbacks的主要作用是：将callback这些节点加入到chain当中去，以便后来的使用

## run_callbacks

~~~rb

	79: def run_callbacks(kind, &block)
    80:   cbs = send("_#{kind}_callbacks")
 => 81:   if cbs.empty?
    82:     yield if block_given?
    83:   else
    84:     runner = cbs.compile
    85:     e = Filters::Environment.new(self, false, nil, block)
    86:     runner.call(e).value
    87:   end
    88: end
~~~

在这里，cbs就是定义了的chain，如果cbs为空，执行方法，如果不为空，那么就需要执行cbs的chain。