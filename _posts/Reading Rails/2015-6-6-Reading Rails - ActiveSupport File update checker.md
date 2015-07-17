---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Reading Rails - File update checker

## File
首先我们来熟悉一下ruby当中对file的处理。
### new
f = File.new("testfile", "r")

在这里，最重要的是熟悉一下r,r+,w+,w,a+,a等用法

~~~rb
"r"  Read-only, starts at beginning of file  (default mode).

"r+" Read-write, starts at beginning of file.

"w"  Write-only, truncates existing file
     to zero length or creates a new file for writing.

"w+" Read-write, truncates existing file to zero length
     or creates a new file for reading and writing.

"a"  Write-only, starts at end of file if file exists,
     otherwise creates a new file for writing.

"a+" Read-write, starts at end of file if file exists,
     otherwise creates a new file for reading and
     writing.
"b"  Binary file.
~~~
我的理解是w+ 跟r+的比较, r＋是从文件最开头开始写，所以是覆盖一些文字，而w＋可以在文件不存在的情况下创建文件，同时将原来文件的内容抹去。
而r+跟w的比较是： w可以创建文件，r+不行，r+可以读文件，而w不行。

### open
File.open("sample.txt", "w") { |file| file.write(str) }

当open函数没有使用block的时候，它的功能跟new是一样的。而当block使用的时候，file作为参数可以传递进block，同时这个文件会自动关闭

### close
### read
### readlines

~~~rb
File.readlines("sample.txt","r").each do |line|
  str_after = str_after + line
end
~~~
### truncate
### write

File.open("sample.txt", "w") { |file| file.write(str) }

## File update checker
接下来我们看FileUpdateChecker
首先我们看一下使用场景，以及我们可以如何使用它

~~~rb

      reloader = ActiveSupport::FileUpdateChecker.new(I18n.load_path.dup){ I18n.reload! }
      app.reloaders << reloader
      ActionDispatch::Reloader.to_prepare do
        reloader.execute_if_updated
        # TODO: remove the following line as soon as the return value of
        # callbacks is ignored, that is, returning `false` does not
        # display a deprecation warning or halts the callback chain.
        true
      end
~~~
从上面我们可以看到FileUpdateChecker的new方法接受了一个路径数组，以及block。
我们来看 new方法。

~~~rb
    def initialize(files, dirs={}, &block)
      @files = files.freeze
      @glob  = compile_glob(dirs)
      @block = block

      @watched    = nil
      @updated_at = nil

      @last_watched   = watched
      @last_update_at = updated_at(@last_watched)
    end
~~~
赋变量值，还有就是watched和updated方法的使用。
watched返回的就是所有存在的files。

我们看看updated_at方法

~~~rb
   def updated_at(paths)
      @updated_at || max_mtime(paths) || Time.at(0)
    end

    def max_mtime(paths)
      time_now = Time.now
      paths.map {|path| File.mtime(path)}.reject {|mtime| time_now < mtime}.max
    end
~~~
max_mtime是所有的路径下面选取修改时间最大的。

接下来我们看看execute_if_updated

~~~rb
    def execute
      @last_watched   = watched
      @last_update_at = updated_at(@last_watched)
      @block.call
    ensure
      @watched = nil
      @updated_at = nil
    end

    # Execute the block given if updated.
    def execute_if_updated
      if updated?
        execute
        true
      else
        false
      end
    end
~~~
在这里先判断有没有update，如果有，则执行execute，也就是执行block语句。

Rails.application.reloaders中存放着需要check的reloader

1. active_support/i18n_railtie.rb
2. Rails::Application::RoutesReloader
3. application/finisher.rb:
