---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Reading Rails - Railties

Railtie 是 Rails 的核心部分之一。通过它，可以扩展和修改 Rails 的初始化程序。

每一个 Rails 组件(如：ActionMailer, ActionController, ActionView 和 ActiveRecord等)都属于 Railtie. 因为它们都需要自己的初始化程序。

什么时候需要使用 Railtie? 当你的扩展符合下列情况时，可以考虑：

替换默认组件
Rails 启动时即要配置内容
Rails 启动时即要初始化内容
Railtie 只是配置及初始化文件

Railtie 不属于真正意义上的”代码”，代码已经完成。想把它运用到 Rails 项目里，并且扩展或修改 Rails 的配置、初始化过程，才需要引进 Railtie.

一个 gem 是 Raitie，通常是指它有 raitie.rb (再准确点，有类继承于 Rails::Railtie) … 并不影响它的其它代码。

通常你的项目代码是单独存放的，raitie.rb 只是针对 Rails 项目初始化或配置工作，不推荐把项目代码放到这里。

Engine 和 Application

Engine 是 Railtie 的子类，所以这里的方法，由于继承关系，它也可以使用。
Application 是 Engine 的子类，所以这里的方法，由于继承关系，它也可以使用。

其它

查看本项目下，所有的 Railtie：

Rails.application.send(:ordered_railties)
Rails 启动是一个复杂的过程，你不必知道具体在哪一步执行 Railtie 代码。

在Railties中有两个重要的类initializable.rb以及configuration.rb
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/lib/rails/initializable.rb
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/lib/rails/railtie/configuration.rb
Initializable 模块顾名思义就是负责初始化，常用的方法只有一个叫initializer的方法，它的方法签名如下，接受一个名字，一个Context，一个可选的参数，一个代码块

	~~~rb
      def initialize(name, context, options, &block)
        options[:group] ||= :default
        @name, @context, @options, @block = name, context, options, block
      end
    ~~~

定义好的 Initializer 代码块会在Rails应用程序启动时执行，并且可以在参数里指定before或者after选项， 让其在某个已定义的Initializer执行之前或之后执行，这个功能是通过Ruby内置的TSort实现的。

Configuration 是实现常见的config.xxx = yyy这一常见写法的源头，它使用了Ruby的method_missing实现了配置参数的属性访问和设置，全部的配置都放在一个名为@@options的类变量里。

	~~~rb
      def method_missing(name, *args, &blk)
        if name.to_s =~ /=$/
          @@options[$`.to_sym] = args.first
        elsif @@options.key?(name)
          @@options[name]
        else
          super
        end
      end
    ~~~

