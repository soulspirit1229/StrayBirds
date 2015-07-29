# Reaing Rails - Railites Configuration

我们首先大致看下configuration配的参数

~~~rb
[32] pry(main)> Rails.application.config.instance_variables.sort
=> [:@allow_concurrency,
 :@assets,
 :@autoflush_log,
 :@autoload_once_paths,
 :@autoload_paths,
 :@beginning_of_week,
 :@cache_classes,
 :@cache_store,
 :@consider_all_requests_local,
 :@eager_load,
 :@eager_load_paths,
 :@encoding,
 :@exceptions_app,
 :@file_watcher,
 :@filter_parameters,
 :@filter_redirect,
 :@force_ssl,
 :@generators,
 :@helpers_paths,
 :@log_formatter,
 :@log_level,
 :@middleware,
 :@paths,
 :@railties_order,
 :@relative_url_root,
 :@reload_classes_only_on_change,
 :@root,
 :@secret_key_base,
 :@secret_token,
 :@serve_static_assets,
 :@session_options,
 :@session_store,
 :@ssl_options,
 :@static_cache_control,
 :@time_zone]
~~~

## generators

~~~rb
=> #<Rails::Configuration::Generators:0x007fc904fc7360
 @aliases={},
 @colorize_logging=true,
 @fallbacks={},
 @hidden_namespaces=["sass"],
 @options=
  {:rails=>
    {:orm=>:active_record,
     :stylesheet_engine=>:scss,
     :javascript_engine=>:coffee,
     :template_engine=>:haml,
     :integration_tool=>:rspec,
     :test_framework=>:rspec},
   :active_record=>{:migration=>true, :timestamps=>true}},
 @templates=["/Users/shenghuan/GitHub/rocket2/lib/templates"]>
~~~

## allow_concurrency

## middleware

~~~rb
[20] pry(main)> Rails.application.config.middleware
=> #<ActionDispatch::MiddlewareStack:0x007fc9043103d8
 @middlewares=
  [Rack::Sendfile,
   ActionDispatch::Static,
   Rack::Lock,
   #<ActiveSupport::Cache::Strategy::LocalCache::Middleware:0x007fc906365048>,
   Rack::Runtime,
   Rack::MethodOverride,
   ActionDispatch::RequestId,
   Rails::Rack::Logger,
   ActionDispatch::ShowExceptions,
   ActionDispatch::DebugExceptions,
   MetaRequest::Middlewares::Headers,
   BetterErrors::Middleware,
   ActionDispatch::RemoteIp,
   ActionDispatch::Reloader,
   ActionDispatch::Callbacks,
   ActiveRecord::Migration::CheckPending,
   ActiveRecord::ConnectionAdapters::ConnectionManagement,
   ActiveRecord::QueryCache,
   ActionDispatch::Cookies,
   ActionDispatch::Session::CookieStore,
   ActionDispatch::Flash,
   ActionDispatch::ParamsParser,
   Rack::Head,
   Rack::ConditionalGet,
   Rack::ETag,
   Rack::Cors,
   Warden::Manager,
   NewRelic::Rack::DeveloperMode,
   NewRelic::Rack::AgentHooks,
   NewRelic::Rack::ErrorCollector,
   Bullet::Rack,
   MetaRequest::Middlewares::MetaRequestHandler,
   MetaRequest::Middlewares::AppRequestHandler,
   ExceptionNotification::Rack]>
~~~