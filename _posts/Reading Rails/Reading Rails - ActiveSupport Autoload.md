# Rails Autoload

ActiveSupport 中使用了很多的autoload，它在Ruby自带的autoload上进行扩展。

1. autoload： 延迟加载
2. eager_autoload eager_load! 预加载


## Autoload源代码
### extended
~~~rb
  module Autoload
    def self.extended(base) # :nodoc:
      base.class_eval do
        @_autoloads = {}
        @_under_path = nil
        @_at_path = nil
        @_eager_autoload = false
      end
    end
~~~
我们来仔细分析一下这个代码，

使用的是extended，就是当其他类extend autoload后需要执行的代码。在这里，它在原来的类上添加class variable.
### autoload
~~~rb
    def autoload(const_name, path = @_at_path)
      unless path
        full = [name, @_under_path, const_name.to_s].compact.join("::")
        path = Inflector.underscore(full)
      end

      if @_eager_autoload
        @_autoloads[const_name] = path
      end

      super const_name, path
    end

~~~

一般我们方法中直接使用autoload

~~~rb
module ActiveSupport
  extend ActiveSupport::Autoload

  autoload :Concern
  autoload :Dependencies
~~~
对照autoload方法，full = ActiveSupport::Concern
而对应生成的path就是activesupport/concern.rb，然后直接调用父类的super方法，这时的super是ActiveSupport里面的super。
所以调用的是Kernel里面的autoload(module,filename)，原生的autoload方法。

### eager_load
那么接下来看eager_load方法

~~~rb
 eager_autoload do
    autoload :BacktraceCleaner
    autoload :ProxyObject
~~~

这里的eager_autoload又调用了autoload方法。

~~~rb
    def eager_autoload
      old_eager, @_eager_autoload = @_eager_autoload, true
      yield
    ensure
      @_eager_autoload = old_eager
    end
~~~
在这里ensure可以单独使用，同时将eager_autoload置为true。在执行上面的autoload的方法的时候，会往@_autoloads这个Hash中放入path路径

然后当你需要显式的调用eager_load时，可以执行

~~~rb
 def eager_load!
      @_autoloads.each_value { |file| require file }
    end
~~~

我们看到我们这里使用的是require就是马上加载。