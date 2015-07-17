# Reading Rails - ActiveSupport Logger

在应用的开发，上线中，我们经常需要查看日志。

ActiveSupport::Logger 继承自Ruby的原生Logger，所以我们先了解一下Ruby 的原生Logger有哪些特性。

## Ruby Logger

当我们创建一个新的Logger的时候

~~~rb
[2] pry(main)> l = Logger.new(STDOUT)
=> #<Logger:0x007ff483605688
 @default_formatter=#<Logger::Formatter:0x007ff483605660 @datetime_format=nil>,
 @formatter=nil,
 @level=0,
 @logdev=
  #<Logger::LogDevice:0x007ff483605610
   @dev=#<IO:/dev/ttys014>,
   @filename=nil,
   @mutex=
    #<Logger::LogDevice::LogDeviceMutex:0x007ff4836055e8
     @mon_count=0,
     @mon_mutex=#<Mutex:0x007ff483605598>,
     @mon_owner=nil>,
   @shift_age=nil,
   @shift_size=nil>,
 @progname=nil>
~~~
我们看到我们可以为log设置formatter,level,file,mutex,progname这些变量.

接下来，我们看ActiveSupport::Logger

## initialize
~~~rb
    def initialize(*args)
      super
      @formatter = SimpleFormatter.new
    end

    # Simple formatter which only displays the message.
    class SimpleFormatter < ::Logger::Formatter
      # This method is invoked when a log event occurs
      def call(severity, timestamp, progname, msg)
        "#{String === msg ? msg : msg.inspect}\n"
      end
    end
~~~

SimpleFormatter仅仅输出msg的信息，而且判断msg是否是String，如果不是String，则调用inspect方法。inspect是Object的方法，它会return string for containing a human-readable repersentation.

## LoggerSilence
ActiveSupport::Logger还include LoggerSilence

~~~rb
module LoggerSilence
  extend ActiveSupport::Concern
  
  included do
    cattr_accessor :silencer
    self.silencer = true
  end

  # Silences the logger for the duration of the block.
  def silence(temporary_level = Logger::ERROR)
    if silencer
      begin
        old_logger_level, self.level = level, temporary_level
        yield self
      ensure
        self.level = old_logger_level
      end
    else
      yield self
    end
  end
end
~~~

cattr_accessor :silencer 相当于给Logger增加了class_variables，并且给其赋值为true。

~~~rb
[29] pry(main)> Rails.logger.class.class_variables
=> [:@@silencer]

[31] pry(main)> Rails.logger.methods.grep /silen/
=> [:silencer,
 :silencer=,
 :silence,
 :silence_warnings,
 :silence_stderr,
 :silence_stream]
~~~

使用silence的方式

~~~rb
def test_silence
  Rails.logger.silence(Logger::ERROR) do;
    Rails.logger.info("this is info, not display");
  end
end
~~~

silence的源代码也比较好看懂，这里就不描述了

## broadcast

~~~rb
    # Broadcasts logs to multiple loggers.
    def self.broadcast(logger) # :nodoc:
      Module.new do
        define_method(:add) do |*args, &block|
          logger.add(*args, &block)
          super(*args, &block)
        end

        define_method(:<<) do |x|
          logger << x
          super(x)
        end

        define_method(:close) do
          logger.close
          super()
        end

        define_method(:progname=) do |name|
          logger.progname = name
          super(name)
        end

        define_method(:formatter=) do |formatter|
          logger.formatter = formatter
          super(formatter)
        end

        define_method(:level=) do |level|
          logger.level = level
          super(level)
        end
      end
    end
~~~

我们可以看到broadcast主要是生成一个module，并且为这个module生成了一些instance methods
[:add, :<<, :close, :progname=, :formatter=, :level=]

~~~rb
file_logger = Logger.new(Rails.root.join("log/alternative-output.log"))
Rails.logger.extend(ActiveSupport::Logger.broadcast(file_logger))
~~~

当其他类比如Rails.logger extend this module，那么就增加了许多class method。
此时我们使用Rails.logger.info "something"会转发到file_logger中. 

我们可以换种写法：

~~~rb
module LoggerProxy

  attr_accessor :logger

  def add (*args, &block)
    @logger.add(*args,&block)
    super(*args, &block)
  end
end

Rails.logger.extend LoggerProxy
Rails.logger.logger =Logger.new(Rails.root.join("log/alternative-output.log"))
~~~

当我们使用Rails.logger.info "something",日志会写在alternative-output.log中

