---
layout: post
title: Reading Rails - Cache
category: Reading Rails
comments: true
---

# Reding Rails - Cache

在Railscast中提到了Rails中的Cache

Rails’ cache store functionality is very modular. It uses the file system to store the cache by default but we can easily change this to store it elsewhere. Rails comes with several cache store options that we can choose from. The default used to be a memory store which stored the cache in local memory of that Rails process. This issue with this is that in production we often have multiple Rails instances running and each of these will have their own cache store which isn’t a good use of resources. The file store works well for smaller applications but isn’t very efficient as reading from and writing to the hard drive is relatively slow. If we use this for a cache that’s accessed frequently we’d be better off using something else.

This brings us to the memcache store which offers the best of both worlds.

## cache_key

cache\_key是model类经常使用的一个方法，它来自于ActiveRecord::Integration中

~~~rb
    def cache_key(*timestamp_names)
      case
      when new_record?
        "#{model_name.cache_key}/new"
      when timestamp_names.any?
        timestamp = max_updated_column_timestamp(timestamp_names)
        timestamp = timestamp.utc.to_s(cache_timestamp_format)
        "#{model_name.cache_key}/#{id}-#{timestamp}"
      when timestamp = max_updated_column_timestamp
        timestamp = timestamp.utc.to_s(cache_timestamp_format)
        "#{model_name.cache_key}/#{id}-#{timestamp}"
      else
        "#{model_name.cache_key}/#{id}"
      end
    end
~~~

cache_key的生成是分情况，如果传入了timestamp，就会根据timestamp进行拼装。
其中 model_name.cache_key就是activemodel naming module中的collections。
如果你的model是LoanApplication,那么

~~~rb
[31] pry(#<LoanApplication>)> self.class.model_name.cache_key
=> "loan_applications"
~~~

而其中的timestamp是根据activerecord下面的timestamp.rb文件中的方法生成的。

~~~rb
    def max_updated_column_timestamp(timestamp_names = timestamp_attributes_for_update)
      if (timestamps = timestamp_names.map { |attr| self[attr] }.compact).present?
        timestamps.map { |ts| ts.to_time }.max
      end
    end
~~~
主要是将model的attribute拿出来之后取最大值。

接下来会讲timestamp转换成字符串，cache_timestamp_format默认为：:nsec。
这样就生成了一个完整的cache_key.

## MemoryStore

我们先来看下memory store的成员变量

~~~rb
def initialize(options = nil)
  options ||= {}
  super(options)
  @data = {}  --存放的健值对，key: :entry
  @key_access = {} 
  @max_size = options[:size] || 32.megabytes --最大内存值32M
  @max_prune_time = options[:max_prune_time] || 2 --
  @cache_size = 0 --cache的容量大小
  @monitor = Monitor.new --同步
  @pruning = false --是否在执行删除cache
end
~~~

memory中存放entry，我们看一个age:12存放的entry是：

ActiveSupport::Cache::Entry:0x007fc9065b3d68 @created_at=1438003969.271652, @expires_in=nil, @value=12

每一个entry都有大小，上面这个entry的容量大小是4，只有当容量达到33554432时，memorystore才会满。粗略的估计一下，可以容纳838万个这么大小的entry。

当我们写入一个value时，Rails.cache.write 'age', 12
那么最重要的方法是write_entry.

~~~rb
def write_entry(key, entry, options) # :nodoc:
  entry.dup_value!
  synchronize do
    old_entry = @data[key]
    return false if @data.key?(key) && options[:unless_exist]
    if old_entry
      @cache_size -= (old_entry.size - entry.size)
    else
      @cache_size += cached_size(key, entry)
    end
    @key_access[key] = Time.now.to_f
    @data[key] = entry
    prune(@max_size * 0.75, @max_prune_time) if @cache_size > @max_size
    true
  end
end
~~~

我们看到如果cache size大于Max size的时候，会执行prune方法。prune方法会将memory删减到最大值的0.75倍。

~~~rb
# To ensure entries fit within the specified memory prune the cache by removing the least
# recently accessed entries.
def prune(target_size, max_time = nil)
  return if pruning?
  @pruning = true
  begin
    start_time = Time.now
    cleanup
    instrument(:prune, target_size, :from => @cache_size) do
      keys = synchronize{ @key_access.keys.sort{|a,b| @key_access[a].to_f <=> @key_access[b].to_f} }
      keys.each do |key|
        delete_entry(key, options)
        return if @cache_size <= target_size || (max_time && Time.now - start_time > max_time)
      end
    end
  ensure
    @pruning = false
  end
end
~~~

它的删除策略是：根据你创建的cache的时间长短。会把创建比较早的删掉。

这个类中使用了个同步用法。

~~~rb
      def synchronize(&block) # :nodoc:
        @monitor.synchronize(&block)
      end
~~~

## FileStore

filestore中主要的write_entry方法。

~~~rb
def write_entry(key, entry, options)
  file_name = key_file_path(key)
  return false if options[:unless_exist] && File.exist?(file_name)
  ensure_cache_path(File.dirname(file_name))
  File.atomic_write(file_name, cache_path) {|f| Marshal.dump(entry, f)}
  true
end
~~~

### atomic_write
atomic_write是为了让你的文件不至于在还没写好的时候就被其他线程操作。

### Marshal

activesupprt 中的marshal是在原生ruby的marshal上做了扩展。

~~~rb
module Marshal
  class << self
    def load_with_autoloading(source)
      load_without_autoloading(source)
    rescue ArgumentError, NameError => exc
      if exc.message.match(%r|undefined class/module (.+)|)
        # try loading the class/module
        $1.constantize
        # if it is a IO we need to go back to read the object
        source.rewind if source.respond_to?(:rewind)
        retry
      else
        raise exc
      end
    end

    alias_method_chain :load, :autoloading
  end
end

~~~