---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Reading Rails - ActiveSupport::MarshalWithAutoloading

## Marshal
我们先来看一下Ruby中的marshall方法

~~~rb
[9] pry(main)> str = Marshal.dump("thing")
=> "\x04\bI\"\nthing\x06:\x06ET"
[10] pry(main)> Marshal.load("\x04\bI\"\nthing\x06:\x06ET")
=> "thing"
[11] pry(main)> Marshal.restore("\x04\bI\"\nthing\x06:\x06ET")
=> "thing"
~~~
Marshal
