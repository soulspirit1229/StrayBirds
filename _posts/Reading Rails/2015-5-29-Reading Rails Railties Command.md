---
layout: post
title: Reading Rails
category: Reading Rails
comments: true
---

# Rails Command

当我们敲下rails g model ..的时候，系统开始调用app下面的rails文件

~~~rb
#!/usr/bin/env ruby
APP_PATH = File.expand_path('../../config/application',  __FILE__)
require_relative '../config/boot'
require 'rails/commands'
~~~

这个时会调用commands.rb，在/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/lib/

~~~rb
rails/commands.rb
ARGV << '--help' if ARGV.empty?
aliases = {
  "g"  => "generate",
  "d"  => "destroy",
  "c"  => "console",
  "s"  => "server",
  "db" => "dbconsole",
  "r"  => "runner"
}
command = ARGV.shift
command = aliases[command] || command
require 'rails/commands/commands_tasks'
Rails::CommandsTasks.new(ARGV).run_command!(command)
~~~


根据你执行的相关命令会调用不同的命令执行

~~~rb
    def run_command!(command)
      command = parse_command(command)
      if COMMAND_WHITELIST.include?(command)
        send(command)
      else
        write_error_message(command)
      end
    end
~~~

当执行generate方法时，调用commands下面的generate.rb
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/lib/rails/commands/

~~~rb
generate.rb
require 'rails/generators'
if [nil, "-h", "--help"].include?(ARGV.first)
  Rails::Generators.help 'generate'
  exit
end
name = ARGV.shift
root = defined?(ENGINE_ROOT) ? ENGINE_ROOT : Rails.root
Rails::Generators.invoke name, ARGV, behavior: :invoke, destination_root: root
然后会调用generator方法执行命令，同理，其他的server, destroy等命令都是执行相应的server.rb  destroy.rb等。
最后会调用/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/lib/rails/generators/rails/model/model_generator.rb
module Rails
  module Generators
    class ModelGenerator < NamedBase # :nodoc:
      argument :attributes, type: :array, default: [], banner: "field[:type][:index] field[:type][:index]"
      hook_for :orm, required: true
    end
  end
end
~~~

 其中hook_for就是调用方法,hook_for在base.rb中
/Users/shenghuan/.rbenv/versions/2.2.1/lib/ruby/gems/2.2.0/gems/railties-4.1.8/lib/rails/generators/base.rb

 ~~~rb
  def self.hook_for(*names, &block)
        options = names.extract_options!
        in_base = options.delete(:in) || base_name
        as_hook = options.delete(:as) || generator_name
        names.each do |name|
          unless class_options.key?(name)
            defaults = if options[:type] == :boolean
              { }
            elsif [true, false].include?(default_value_for_option(name, options))
              { banner: "" }
            else
              { desc: "#{name.to_s.humanize} to be invoked", banner: "NAME" }
            end
            class_option(name, defaults.merge!(options))
          end
          hooks[name] = [ in_base, as_hook ]
          invoke_from_option(name, options, &block)
        end
~~~


