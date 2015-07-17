# Reading Rails - ActiveSupport Array

首先我们来看core_ext/array下面的文件
在access中比较简单，方法如下
from
to
without
second
third
fourth
fifth
forty_two

## to_sentence

接下来，我们来看conversions.rb
1 to_sentence的用法
options.assert_valid_keys(:words_connector, :two_words_connector, :last_word_connector, :locale)
Hash的assert_valid_keys是core_ext中引入的，可以进行校验参数
  def assert_valid_keys(*valid_keys)
    valid_keys.flatten!
    each_key do |k|
      unless valid_keys.include?(k)
        raise ArgumentError.new("Unknown key: #{k.inspect}. Valid keys are: #{valid_keys.map(&:inspect).join(
valid_keys是个Array，Array中的flattern的语法
a = [ 1, 2, [3, [4, 5] ] ]
a.flatten!   #=> [1, 2, 3, 4, 5]
所以我们看assert_valid_keys中如果valid_keys中不能包含Hash中的key，那么就会抛出ArgumentError。
如果valid_keys是个空数组，那么不会进入到each_key的循环中，所以
{}.assert_valid_keys(:name,:years,:age) 为true
接下来就是取I18n中对应的words_connector，然后进行拼装字符串.

## to_formatted_s 

2 to_formatted_s
  def to_formatted_s(format = :default)
    case format
    when :db
      if empty?
        'null'
      else
        collect(&:id).join(',')
      end
    else
      to_default_s
    end
  end
  alias_method :to_default_s, :to_s
  alias_method :to_s, :to_formatted_s

有两个知识点，
A首先这个方法extends了Array的to_s方法
通过使用alias_method，所以可以用[].to_s(:db)的方法
B alias_method
alias跟alias_method
alias(p1,p2)的第一个参数是新方法，第二个参数是原来的方法，它的本意就是让新方法代替原来的方法。

class Microwave
  def on
    puts "The microwave is on"
  end
  alias :start :on
end
m = Microwave.new
m.start # same as m.on

alias_method跟alias相似，参数是new method name, old method name，中间用逗号隔开

## to_xml

## extract_options!
  def extract_options!
    if last.is_a?(Hash) && last.extractable_options?
      pop
    else
      {}
    end
  end

Array的pop method是将最后一个元素pop out。所以extract_options有两个结果：
A：最后得到的结果是最后一个hash元素。
B：同时Array本身的options也只剩下前面的元素。

## in\_groups_of

  def in_groups_of(number, fill_with = nil)
    if number.to_i <= 0
      raise ArgumentError,
        "Group size must be a positive integer, was #{number.inspect}"
    end
    if fill_with == false
      collection = self
    else
      # size % number gives how many extra we have;
      # subtracting from number gives how many to add;
      # modulo number ensures we don't add group of just fill.
      padding = (number - size % number) % number
      collection = dup.concat(Array.new(padding, fill_with))
    end
    if block_given?
      collection.each_slice(number) { |slice| yield(slice) }
    else
      collection.each_slice(number).to_a
    end
  end
这个方法是在Enumerable的each_slice的基础上进行封装的。默认fill_with是nil的值，所以不传fill_with的时候，slice中会自动用nil补齐，如果不想使用fill_with=false

A dup的使用
    dup是Object的method，是shallow copy

B concat的使用
concat是两个数组合并

C each_slice是Enumerable的方法，each_slice的返回结果是Enumrator

D block_given?的使用，使得语法更具有灵活性

## in groups

## split
  def split(value = nil)
    if block_given?
      inject([[]]) do |results, element|
        if yield(element)
          results << []
        else
          results.last << element
        end
        results
      end
    else
      results, arr = [[]], self.dup
      until arr.empty?
        if (idx = arr.index(value))
          results.last.concat(arr.shift(idx))
          arr.shift
          results << []
        else
          results.last.concat(arr.shift(arr.size))
        end
      end
      results
    end
  end

A shift的使用，二维数组的创建
二维数组的创建[[]]

arr.shift有两个作用
a. 返回的数组是左移的数组
b. arr自身只留下左移之后的数组

B block_given的使用

## inquiry

## prepend_and_append
class Array
  # The human way of thinking about adding stuff to the end of a list is with append.
  alias_method :append,  :<<
  # The human way of thinking about adding stuff to the beginning of a list is with prepend.
  alias_method :prepend, :unshift
end

通过使用alias_method实现方法的重命名

## wrap
def self.wrap(object)
    if object.nil?
      []
    elsif object.respond_to?(:to_ary)
      object.to_ary || [object]
    else
      [object]
    end
  end

A respond_to?的用法
判断该方法是否有
B 在rails中使用[account: [profile: :name]]
[{:account=>[{:profile=>:name}]}]