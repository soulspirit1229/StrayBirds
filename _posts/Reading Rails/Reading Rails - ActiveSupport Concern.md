# Reading Rails - ActiveSupport Concern
Rails concern可以使代码写的更简单，在它的comment中做了很好的解释。

### included

要了解included的功能，我们先要了解ClassMethods, self.included, class_eval等语法。
我们写class methods和 instance methods，可以有不同的写法。

~~~rb
module M
  def self.x
  end

  def y
  end
end

is equal to 

module M
  module ClassMethods
    def x
    end
  end
  module InstanceMethods
    def y
    end
  end
end
~~~
为什么要有这么多的写法呢？是不是有些场合使用self.xx，有些场合使用ClassMethods更合理些？

我们清楚了ClassMethods的用法，接下来我们看class_eval的方法

~~~rb
class Admin << ActiveRecord::Base
	has_many :profile
end

is equal to 

Admin.class_eval do
	has_many :profile
end
~~~

理解了上面的，我们再看一段代码

~~~rb
module M
  def self.included(base)
    base.extend ClassMethods
    base.class_eval do
      p "something happened"
      def do_something_in_included
        p "do something in included"
      end
    end
  end

  module ClassMethods
    def do_something_in_M
      p "do something in M"
    end
  end
end

class N
  include M

end

n = N.new
n.do_something_in_included
N.do_something_in_M

~~~

在module M中，我们定义了included方法，以及ClassMethods。class_eval就是往里面增加instance methods。 而
base.extend ClassMethods是增加了Class Methods。所以通过引入include M, N增加了两个方法，一个实例方法，一个类方法。

更近一步，我们可以将上面的M进行简化

~~~rb
module M
  extend ActiveSupport::Concern

  included do
    p "something happened"

    def do_something_in_included
      p "do something in included"
    end
  end

  module ClassMethods
    def do_something_in_M
      p "do something in M"
    end
  end
end
~~~
这个M extend Concern，实现的功能跟上面一样。

Ok，了解了concern的作用，接下来开始看源代码：

~~~rb
  def included(base = nil, &block)
      if base.nil?
        raise MultipleIncludedBlocks if instance_variable_defined?(:@_included_block)

        @_included_block = block
      else
        super
      end
    end
~~~


### append_features
Ruby的Module class有append\_features这个方法， 当这个module被其他的类include的时候，Ruby会调用append_features。 Ruby本身的实现方式是把被included的constants,methods,以及module variables传入到include module.

rails中的concern重新定义了这个方法。

~~~rb
    def append_features(base)
      if base.instance_variable_defined?(:@_dependencies)
        base.instance_variable_get(:@_dependencies) << self
        return false
      else
        return false if base < self
        @_dependencies.each { |dep| base.include(dep) }
        super
        base.extend const_get(:ClassMethods) if const_defined?(:ClassMethods)
        base.class_eval(&@_included_block) if instance_variable_defined?(:@_included_block)
      end
    end
~~~

我们用个例子来说明

~~~rb
module M
  extend ActiveSupport::Concern

  included do
    p "something happened"

    def do_something_in_included
      p "do something in included"
    end
  end

  module ClassMethods
    def do_something_in_M
      p "do something in M"
    end
  end
end

module MM
  extend ActiveSupport::Concern
  include M

  included do
    def do_method_in_mm
      p "method in mm"
    end
  end

end

class MMM
  include MM

end

mmm = MMM.new

[7] pry(main)> mmm.methods.grep /do/
=> [:do_something_in_included, :do_method_in_mm]
[8] pry(main)> MMM.methods.grep /do/
=> [:do_something_in_M]
[9] pry(main)> MMM.instance_variables
=> []
[10] pry(main)> MM.instance_variables
=> [:@_dependencies, :@_included_block]
[11] pry(main)> MM.instance_variable_get(:@_dependencies)
=> [M]
[12] pry(main)> MM.instance_variable_get(:@_included_block)
=> #<Proc:0x007fdcea3dcb50@(pry):22>
[13] pry(main)> M.instance_values
=> {"_dependencies"=>[], "_included_block"=>#<Proc:0x007fdcf0f3d408@(pry):4>}
~~~

我们可以看到在MM include M的时候，会执行append_features,此时 base.instance_variable_defined?(:@_dependencies)为true， 所以不会执行默认的ruby的append_features.

当MMM include MM的时候， 此时 base.instance_variable_defined?(:@_dependencies) 为false，此时执行的就是else中的语句。

@_dependencies.each { |dep| base.include(dep) }此时会将M included到 MMM中，然后调用super,来调用Ruby默认的行为。同时extend ClassMethods and includes InstanceMethods.最后执行class_eval，在这里，执行的就是 MM.instance_variable_get(:@_included_block) 这个proc。至此，所有的东西都放入到了MMM中

### class_methods
最新的 Rails中还增加了class_methods

~~~rb
    def class_methods(&class_methods_module_definition)
      mod = const_defined?(:ClassMethods) ?
        const_get(:ClassMethods) :
        const_set(:ClassMethods, Module.new)

      mod.module_eval(&class_methods_module_definition)
    end
~~~
