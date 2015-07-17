# Reaidng Rails - Rails的启动过程


[链接一](http://www.tuicool.com/articles/2meEze)

[链接二](http://meladet.iteye.com/blog/423452)

## Rails执行环境
在ruby的bin文件执行环境下面
~/.rbenv/versions/2.2.1/bin/rails

~~~rb
require 'rubygems'
version = ">= 0"
if ARGV.first
  str = ARGV.first
  str = str.dup.force_encoding("BINARY") if str.respond_to? :force_encoding
  if str =~ /\A_(.*)_\z/ and Gem::Version.correct?($1) then
    version = $1
    ARGV.shift
  end
end
gem 'railties', version
load Gem.bin_path('railties', 'rails', version)
~~~

先解释一下这这个文件中的内容，#开头的注释不多说： 
1、require‘rubygems’,这句话说明需要加载rubygems.rb文件，那么这个文件在什么地方呢？这边需要了解require加载文件时候的查找路径都有哪些，ruby中也有类似java classpath的一个东西叫做$LOAD_PATH，存储在$:中，require加载后面的文件就是从这个变量中的路径开始。打开命令行窗口，输入ruby –e “puts $:”或者 ruby –e ”puts $LOAD_PATH”可以查看你系统中的ruby的loadpath。

~~~rb
~ ➤ ruby -e "puts $:"
/Users/shenghuan/.rbenv/rbenv.d/exec/gem-rehash
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/site_ruby/2.2.0
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/site_ruby/2.2.0/x86_64-darwin14
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/site_ruby
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/vendor_ruby/2.2.0
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/vendor_ruby/2.2.0/x86_64-darwin14
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/vendor_ruby
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/2.2.0
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/2.2.0/x86_64-darwin14
~~~

那么require ’rubygems’它就会去上面列出的几个路径中查找一个文件名为rubygems.rb，不难发现在/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/2.2.0
下面有这个文件。
 
2、version = "> 0"，这句话说明加载gem的默认版本 

3、这一段if可以接收用户输入gem的版本，如果不输入，则默认加载最新的版本。 

4、gem ‘railties’,version调用的rubygems.rb中的方法，方法定义如下，这个方法的作用是激活满足版本条件的rails 
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/2.2.0/rubygems.rb

5、到最后一句load ‘rails’了，这句话其实跟reqiure的功能一样。只不过是每次都重新加载rails文件，这个‘rails’文件也是从$LOAD_PATH中查找，在上一点说过Gem.activate(gem_name, *version_requirements)这个方法最后动态插入了gem_name相关的bin_path到$LOAD_PATH中，这个bin_path就包含了我们要找的’rails’。

在rails中添加puts Gem.bin_path('railties','rails',version) ，打印出Rails的执行路径，重新执行Rails命令
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/bin/rails
我们转到/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/bin/rails目录下，用文本编辑器打开

~~~rb
#!/usr/bin/env ruby
git_path = File.expand_path('../../../.git', __FILE__)
if File.exist?(git_path)
  railties_path = File.expand_path('../../lib', __FILE__)
  $:.unshift(railties_path)
end
require "rails/cli"
加载/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/lib下的文件
rails/cli的源代码如下
require 'rails/app_rails_loader'
# If we are inside a Rails application this method performs an exec and thus
# the rest of this script is not run.
Rails::AppRailsLoader.exec_app_rails
require 'rails/ruby_version_check'
Signal.trap("INT") { puts; exit(1) }
if ARGV.first == 'plugin'
  ARGV.shift
  require 'rails/commands/plugin'
else
  require 'rails/commands/application'
end
首先check ruby的version，然后根据敲的命令是否有plugin，在调取相应的命令。如果是rails new app，那么执行'rails/commands/application'，代码如下
require 'rails/generators'
require 'rails/generators/rails/app/app_generator'
module Rails
  module Generators
    class AppGenerator # :nodoc:
      # We want to exit on failure to be kind to other libraries
      # This is only when accessing via CLI
      def self.exit_on_failure?
        true
      end
    end
  end
end
args = Rails::Generators::ARGVScrubber.new(ARGV).prepare!
Rails::Generators::AppGenerator.start args
~~~

对参数做完处理后，使用AppGenerator启动
