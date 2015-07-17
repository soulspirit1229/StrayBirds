---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Rails Lazily Load Hooks

ActiveSupport中

## LazyLoadHooks源代码
### initializer
~~~rb
initializer 'active_record.initialize_timezone' do
  ActiveSupport.on_load(:active_record) do
    self.time_zone_aware_attributes = true
    self.default_timezone = :utc
  end
end
~~~
首先调用了initializer方法，该方法是在initializable.rb中

~~~rb
      def initializer(name, opts = {}, &blk)
        raise ArgumentError, "A block must be passed when defining an initializer" unless blk
        opts[:after] ||= initializers.last.name unless initializers.empty? || initializers.find { |i| i.name == opts[:before] }
        initializers << Initializer.new(name, nil, opts, &blk)
      end
~~~
这个我们将在之后的文章中分析。

load hook在Rails中的意思是当程序在启动的时候，有一些配置需要应用到相应的module上，在这时，active_record是没有被加载的。当:active_record加载时，将这些配置应用到active record上面。

我们明白了这段代码之后，我们接下来看on_load方法。

### on_load
~~~rb
  @load_hooks = Hash.new { |h,k| h[k] = [] }
  @loaded = Hash.new { |h,k| h[k] = [] }

  def self.on_load(name, options = {}, &block)
    @loaded[name].each do |base|
      execute_hook(base, options, block)
    end

    @load_hooks[name] << [block, options]
  end
~~~
首先新建了两个Hash，我们可以在Rails console中查看这两个值存储了哪些value。

~~~rb
[7] pry(main)> ActiveSupport.instance_variables
=> [:@load_hooks, :@loaded, :@_autoloads, :@_under_path, :@_at_path, :@_eager_autoload]

[11] pry(main)> ActiveSupport.instance_variable_get(:@loaded)
=> {:i18n=>[Object],
 :after_initialize=>
  [#<Rocket2::Application:0x007fdcea2abb50
    @_all_autoload_paths=
     ["/Users/shenghuan/GitHub/rocket2/lib",
      "/Users/shenghuan/GitHub/rocket2/lib/yilian",
      "/Users/shenghuan/GitHub/rocket2/app/gateways",
      "/Users/shenghuan/GitHub/rocket2/lib/validators",

[12] pry(main)> ActiveSupport.instance_variable_get(:@load_hooks)
=> {:i18n=>
  [[#<Proc:0x007fdce8937980@/Users/shenghuan/.rbenv/versions/2.0.0-p598/lib/ruby/gems/2.0.0/gems/activemodel-4.1.8/lib/active_model.rb:69>,
    {}],
   [#<Proc:0x007fdce8b54a38@/Users/shenghuan/.rbenv/versions/2.0.0-p598/lib/ruby/gems/2.0.0/gems/activerecord-4.1.8/lib/active_record.rb:169>,
    {}],
   [#<Proc:0x007fdce8aa76f8@/Users/shenghuan/.rbenv/versions/2.0.0-p598/lib/ruby/gems/2.0.0/gems/actionview-4.1.8/lib/action_view.rb:95>,
    {}]],
 :after_initialize=>
  [[#<Proc:0x007fdcea11ade0@/Users/shenghuan/.rbenv/versions/2.0.0-p598/lib/ruby/gems/2.0.0/gems/activesupport-4.1.8/lib/active_suppo
~~~

这里的关键是execute_hook method，是它给ActiveRecord::Base中的attribute进行赋值的

### execute_hook

~~~rb
  def self.execute_hook(base, options, block)
    if options[:yield]
      block.call(base)
    else
      base.instance_eval(&block)
    end
  end
~~~

其中base就是ActiveRecord::Base，而instance_eval就是根据block中的方法为其属性赋值。

根据对代码的理解，我们可以写一个简单的initializers

~~~rb
module Base
  def self.time_zone
    @time_zone
  end

  def self.time_zone=(time_zone)
    @time_zone = time_zone
  end

end

def initializers(name, &block)
  p name
  execute_hook(Base, block)
end

def execute_hook(base, block)
  base.instance_eval(&block)
end


initializers "add time zone for base" do
  self.time_zone = :utc
end
p Base.time_zone
~~~

### run load hooks

~~~rb
  def self.run_load_hooks(name, base = Object)
    @loaded[name] << base
    @load_hooks[name].each do |hook, options|
      execute_hook(base, options, hook)
    end
  end
~~~
还不是很理解为什么要执行两遍execute_hook的方法。

