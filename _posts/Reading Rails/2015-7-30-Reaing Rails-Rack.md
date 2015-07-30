---
layout: post
title: Reading Rails - Rack
category: Reading Rails
comments: true
---

# Reaing Rails - Rack

今天在耳语上看到一段话说：
深圳是个大赌场，赌输的男人走了，赌赢的男人留了下来，还有的男人继续在搏。它是男人的天堂，又是男人的地狱，男人们在这里承受最大的压力，又享受最大的自由，男人们从一个极端到另一个极端，终于找到了自己，然后又失去了自己。

感觉写的很不错，深圳的创业氛围很浓，如果这里有初创企业，也请联系我，让我们为下半辈子成为“废人”而努力。

进入正题。
我们知道Rack连接了Web Server和Web App，它是个middleware，同时也可以是一个最小的web app。

最简单的web.就是创建个config.ru, 里面写上代码

~~~rb
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }
~~~

执行rakeup config.ru，最简单的web server产生了。

那么Rails系统当中一般会使用哪些middleware呢？它们又是如何在Rails系统中一步步执行的呢？

首先我们来看有哪些middleware 我们可以执行bundle exec rake middleware或者在console中执行Rails.application.config.middleware。

~~~rb
use Rack::Sendfile
use ActionDispatch::Static
use Rack::Lock
use #<ActiveSupport::Cache::Strategy::LocalCache::Middleware:0x007ff45aaa90f0>
use Rack::Runtime
use Rack::MethodOverride
use ActionDispatch::RequestId
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use ActionDispatch::DebugExceptions
use MetaRequest::Middlewares::Headers
use BetterErrors::Middleware
use ActionDispatch::RemoteIp
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::Migration::CheckPending
use ActiveRecord::ConnectionAdapters::ConnectionManagement
use ActiveRecord::QueryCache
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use ActionDispatch::ParamsParser
use Rack::Head
use Rack::ConditionalGet
use Rack::ETag
use Rack::Cors
use Warden::Manager
use Bullet::Rack
use MetaRequest::Middlewares::MetaRequestHandler
use MetaRequest::Middlewares::AppRequestHandler
use ExceptionNotification::Rack
run Rocket2::Application.routes
~~~

## 默认middleware的加载
知道了rails的加载结果，我们可以探究的更深，就是Rails是如何加载这些middleware的。
Rails在启动过程中，会调用Engine下的app方法。

~~~rb
    # Returns the underlying rack application for this engine.
    def app
      @app ||= begin
        config.middleware = config.middleware.merge_into(default_middleware_stack)
        config.middleware.build(endpoint)
      end
    end
~~~

default_middleware_stack方法就是加载了默认的middleware，具体的代码在railties下面DefaultMiddlewareStack中build_stack方法。

~~~rb
middleware.use ::ActionDispatch::ParamsParser
middleware.use ::Rack::Head
middleware.use ::Rack::ConditionalGet
~~~

而每个middleware又是如何串在一起的呢？config.middleware.build(endpoint)实现了这个功能.
在ActionDispatch::MiddlewareStack中的build方法阐述了rails是如何将这些middleware连在一起的。

~~~rb
   def build(app = nil, &block)
      app ||= block
      raise "MiddlewareStack#build requires an app" unless app
      middlewares.freeze.reverse.inject(app) { |a, e| e.build(a) }
    end
~~~
通过inject方法，将app一层层套在一起。

直白点解释就是：

~~~rb
m1 = middlewares.freeze.reverse[0]
ExceptionNotification::Rack
m1f = m1.build(app)
<ExceptionNotification::Rack:0x007fd2a22326d0 @app=#<ActionDispatch::Routing::RouteSet:0x007fd2a0e087e0>>
m2 = middlewares.freeze.reverse[1]
MetaRequest::Middlewares::AppRequestHandler
m2f = m2.build(m1f)
<MetaRequest::Middlewares::AppRequestHandler:0x007fd2a22336e8 @app=#<ExceptionNotification::Rack:0x007fd2a22326d0 @app=#<ActionDispatch::Routing::RouteSet:0x007fd2a0e087e0>>>
~~~


那么这些串起来的middleware是怎么执行的呢？

## middleware的执行

我们以ActionDispatcher::ParamsParser为例，

~~~rb
    def initialize(app, parsers = {})
      @app, @parsers = app, DEFAULT_PARSERS.merge(parsers)
    end

    def call(env)
      if params = parse_formatted_parameters(env)
        env["action_dispatch.request.request_parameters"] = params
      end

      @app.call(env)
    end
~~~

从上文知道，@app是下一个middleware 模块。Rails在将params格式处理之后，就执行下一个middleware的call方法。
最后走到ActionDispatch::Routing::RouteSet的call方法

~~~rb
      def call(env)
        @router.call(env)
      end
~~~

走到ActionDispatch::Journey::Router的call方法。

~~~rb
      def call(env)
        env['PATH_INFO'] = Utils.normalize_path(env['PATH_INFO'])

        find_routes(env).each do |match, parameters, route|
          script_name, path_info, set_params = env.values_at('SCRIPT_NAME',
                                                             'PATH_INFO',
                                                             @params_key)

          unless route.path.anchored
            env['SCRIPT_NAME'] = (script_name.to_s + match.to_s).chomp('/')
            matched_path = match.post_match
            env['PATH_INFO']   = matched_path
            env['PATH_INFO']   = "/" + matched_path unless matched_path.start_with? "/"
          end

          env[@params_key] = (set_params || {}).merge parameters
          status, headers, body = route.app.call(env)

          if 'pass' == headers['X-Cascade']
            env['SCRIPT_NAME'] = script_name
            env['PATH_INFO']   = path_info
            env[@params_key]   = set_params
            next
          end

          return [status, headers, body]
        end

        return [404, {'X-Cascade' => 'pass'}, ['Not Found']]
      end
~~~
到这里就接着调用dispatcher，然后调用我们controller的方法执行请求了。


## middleware的添加删除

我们知道我们可以通过添加，删除middleware。

config.middleware.use Rack::BounceFavicon

上面就添加了一个middleware，那么Rails是如何添加这个middleware的呢？
config.middleware是 Rails::Configuration::MiddlewareStackProxy类的实例。我们来看看use方法。

~~~rb
def use(*args, &block)
  @operations << [__method__, args, block]
end

def merge_into(other) #:nodoc:
  @operations.each do |operation, args, block|
    other.send(operation, *args, &block)
  end
  other
end
~~~

后续执行merge_into方法的时候,是调用传入参数other的方法。所以Proxy的use其实调用的是ActionDispatcher::MiddlewareStack.
其中middlewares就是个数组。用use，就是push到array中。也就是说Rails维护一个middleware的数组，在初始化的时候通过读取environment.rb的方法来生成最终的middleware数组。

~~~rb
def insert(index, *args, &block)
  index = assert_index(index, :before)
  middleware = self.class::Middleware.new(*args, &block)
  middlewares.insert(index, middleware)
end

alias_method :insert_before, :insert

def insert_after(index, *args, &block)
  index = assert_index(index, :after)
  insert(index + 1, *args, &block)
end

def use(*args, &block)
  middleware = self.class::Middleware.new(*args, &block)
  middlewares.push(middleware)
end
~~~

## 常用的rack

接下来会以ExceptionNotification::Rack这个组件说明我们可以通过middleware所做的事。

未完待续。

