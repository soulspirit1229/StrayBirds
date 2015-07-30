# Reading Ruby - Struct, OpenStruct

struct和openstruct都在ruby的源代码中，

一般我们可以这么使用struct

~~~rb
Customer = Struct.new(:name, :address) do
  def greeting
    "Hello #{name}!"
  end
end
Customer.new("Dave", "123 Main").greeting  # => "Hello Dave!"
~~~

但是如果我们使用了未定义的属性，struct会报错。

~~~rb
customer = Customer.new("Dave", "123 Main")
customer.id --报错
~~~

但是openstruct可以随时动态增加属性。

~~~rb
person = OpenStruct.new('name' => 'John Smith', 'age' => 70)
person[:age] # => 70, same as ostruct.age

person.sex = 'male'
~~~

我们来看看openstruct的源码，首先是initialize方法

~~~rb
  def initialize(hash=nil)
    @table = {}
    if hash
      hash.each_pair do |k, v|
        k = k.to_sym
        @table[k] = v
        new_ostruct_member(k)
      end
    end
  end
~~~

内部使用的是个hash @table。把传入的hash值赋值给@table，同时执行new_ostruct_member方法。

~~~rb
 def new_ostruct_member(name)
    name = name.to_sym
    unless respond_to?(name)
      define_singleton_method(name) { @table[name] }
      define_singleton_method("#{name}=") { |x| modifiable[name] = x }
    end
    name
  end
~~~

new_ostruct_member主要是添加了两个方法， read key and write key.
读取的时候主要从@table中读取，但是写入用到了modifiable。modifiable返回的也是@table，不过不清楚这里为什么要调用modifiable方法。

接下来当我们调用 person.sex的方法的时候，调用了openstruct的methods_missing方法。

~~~rb
  def method_missing(mid, *args) # :nodoc:
    mname = mid.id2name
    len = args.length
    if mname.chomp!('=')
      if len != 1
        raise ArgumentError, "wrong number of arguments (#{len} for 1)", caller(1)
      end
      modifiable[new_ostruct_member(mname)] = args[0]
    elsif len == 0
      @table[mid]
    else
      err = NoMethodError.new "undefined method `#{mid}' for #{self}", mid, args
      err.set_backtrace caller(1)
      raise err
    end
  end
~~~

比较简单，根据传入的方法是否含有＝进行判断，有等号的就给@table中添加，没有等号的直接读取@table中的值。


