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

