# Reaing Rails - Rack

我们知道Rack连接了Web Server和Web App，它是个middleware，同时也是一个最小的web app。

最简单的web.就是创建个config.ru, 里面写上代码

~~~rb
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }
~~~

执行rakeup config.ru，最简单的web server产生了。

那么我们系统当中一般会使用哪些middleware呢？它们又是如何一步步执行的呢？

我们可以执行bundle exec rake middleware或者在console中执行Rails.application.config.middleware。

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
在railties下面DefaultMiddlewareStack中build_stack方法加载了默认的middleware

~~~rb
middleware.use ::ActionDispatch::ParamsParser
middleware.use ::Rack::Head
middleware.use ::Rack::ConditionalGet
~~~

那么这些串起来的middleware是怎么执行的呢？

## middleware的执行

