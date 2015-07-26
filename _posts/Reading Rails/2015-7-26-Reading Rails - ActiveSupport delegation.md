---
layout: post
title: Reading Rails - ActiveSupport delegation
category: Reading Rails
comments: true
---

# Reading Rails - ActiveSupport delegation

我们来看个例子。

~~~rb
class Foo
  def initialize(bar)
    @bar = bar
  end

  delegate :name, to: :@bar, allow_nil: true
end
~~~

我们看delegation的源代码，最重要的是生成方法那一段。

~~~rb
if allow_nil
  method_def = [
    "def #{method_prefix}#{method}(#{definition})",
    "_ = #{to}",
    "if !_.nil? || nil.respond_to?(:#{method})",
    "  _.#{method}(#{definition})",
    "end",
  "end"
  ].join ';'
else
  exception = %(raise DelegationError, "#{self}##{method_prefix}#{method} delegated to #{to}.#{method}, but #{to} is nil: \#{self.inspect}")

  method_def = [
    "def #{method_prefix}#{method}(#{definition})",
    " _ = #{to}",
    "  _.#{method}(#{definition})",
    "rescue NoMethodError => e",
    "  if _.nil? && e.name == :#{method}",
    "    #{exception}",
    "  else",
    "    raise",
    "  end",
    "end"
  ].join ';'
end
~~~

在这里，因为prefix为空，而且allow_nil,那么最终会生成的方法是：

~~~rb
def name(*args, &block);
  _ = @bar
  if !_.nil? || nil.respond_to?(:name);
    _.name(*args, &block)
  end
end"
~~~

这个方法比较简单。就是分发方法。
