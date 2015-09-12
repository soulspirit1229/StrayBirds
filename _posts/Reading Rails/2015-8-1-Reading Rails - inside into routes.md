# Reading Rails - inside into routes

## 前言
今天刚去看了破风，记录两句话： 风在前，无惧。但同时人生不应该为了赢而搞乱自己。

## Tips
先介绍个小技巧，当我们在判断一个路径是否能match到我们自己编写的controller的时候，我们可以使用下面的方法：
在Rails console中使用recoginzie_path方法。

~~~rb
[7] pry(main)> app._routes.recognize_path("http://localhost:3000/ninja/users/11")
=> {:action=>"show", :controller=>"admins/users", :id=>"11"}
~~~

## 源代码解读

Route的代码并不好读，第一个原因是相比其他的模块，route的注释较少。第二个原因是：约定先于配置使得代码需要处理的情况比较多，同时路径的匹配算法也不容易理解。但我们今天还是通过尝试解读源码来理解

1. routes.rb是怎么起作用的
2. request请求又是如何与controller相匹配的

### Routes的产生
首先来看个例子，用代码来实现这样的输入输出

~~~rb
# hash = config do
#   namespace :users do
#     config :session_timeout, 30
#     config :minimum_password_length, 20
#     namespace :nickname do
#             config :default, "Nick"
#             config :max_length, 50
#     end
#   end
# end
#
# { “users.session_timeout”: 30, “users.minimum_password_length”: 20, “users.nickname.max_length”: 50, “users.nickname.default”: “Nick”}
~~~

结果如下：

~~~rb
def config(&block)
  eval_block(&block)
end

def eval_block(&block)
  mapper = Mapper.new
  mapper.instance_exec(&block)
  p mapper.config_hash
end

class Scope

  def initialize(name,parent,scope_level)
    @name = name.to_s
    @parent = parent
    @scope_level = scope_level
  end

  def name
    if @parent && @parent.name != ""
      "#{@parent.name}.#{@name}"
    else
      @name
    end
  end
end

class Mapper

  attr_accessor :config_hash

  def initialize
    @scope = Scope.new("",nil,"")
    @config_hash = {}
  end

  def namespace(name, &block)
    old, new_scope = @scope, Scope.new(name, @scope, "namespace")
    @scope = new_scope
    # apply_behaiver_for(name, new_scope, &block)

    yield if block_given?
    self
  ensure
    @scope = old
  end

  def config(*options)
    merge_config_name(options[0], options[1])
  end

  def merge_config_name(key,value)
    config_hash[@scope.name + "." + key.to_s] = value
  end
end
~~~


大家也发现了这个跟routes的实现很像，要想理解Rails中Routes的生成，首先这个看懂就比较好深入的了解routes的实现原理了。
我们以最简单的routes.rb为例来开始分析：

~~~rb
Rails.application.routes.draw do
  root to: 'home#index'
  ...
~~~

这里的Rails.application.routes 调用的是Engine中的routes方法

~~~rb
# Defines the routes for this engine. If a block is given to
# routes, it is appended to the engine.
def routes
  @routes ||= ActionDispatch::Routing::RouteSet.new
  @routes.append(&Proc.new) if block_given?
  @routes
end
~~~

Rails.application.routes就是Routing::RouteSet, 一个Application包含一个RouteSet，一个RouteSet包含N个Route，一个Route就代表一个路由信息。接下来调用RouteSet中的draw方法。

~~~rb
def draw(&block)
  clear! unless @disable_clear_and_finalize
  eval_block(block)
  finalize! unless @disable_clear_and_finalize
  nil
end

def eval_block(block)
  ...
  mapper = Mapper.new(self)
  if default_scope
    mapper.with_default_scope(default_scope, &block)
  else
    mapper.instance_exec(&block)
  end
end
~~~
draw方法中比较重要的是eval\_block方法。而例子中 root to: 'home#index' 作为block方法传入到这个方法中。eval\_block(block)的这句代码带入了routing中最重要的Mapper类。eval\_block方法最终执行的是mapper.instance_exec(&block)，定义在routes.rb中的方法是由Mapper这个类来负责解析。我们来看看这个重要的Mapper类。

### Mapper

1. 解析routes.rb中的block，生成代表路由的Route
2. 最重要的方法：add_route，定义在routes.rb中的resource,match,get,post等方法最终执行的都是这个方法。

~~~rb
module ActionDispatch
  module Routing
    class Mapper
      ...
      include Base
      include HttpHelpers
      include Redirection
      include Scoping
      include Concerns
      include Resources
~~~

我们先查看Mapping 的祖先链

~~~rb
Mapping.ancestors:
[ActionDispatch::Routing::Mapper, ActionDispatch::Routing::Mapper::Resources, ActionDispatch::Routing::Mapper::Concerns, ActionDispatch::Routing::Mapper::Scoping, ActionDispatch::Routing::Redirection, ActionDispatch::Routing::Mapper::HttpHelpers, ActionDispatch::Routing::Mapper::Base, Object, PP::ObjectMixin, Delayed::MessageSending, ActiveSupport::Dependencies::Loadable, JSON::Ext::Generator::GeneratorMethods::Object, Kernel, BasicObject]
~~~

Mapper include这些module，比如Base中定义了3个方法: root,match,mount.而HttpHelpers定义了get,post等方法。这些方法就是routes中使用的常用方法。所以如果我们通过打开Mapper类，定义方法，那么routes.rb中就可以使用自己定义的方法。以devise为例，devise定义了devise_for方法。因此routes.rb中可以使用devise\_for 方法。

~~~rb
module ActionDispatch::Routing
  class RouteSet
    class Mapper
      def devise_for(*resources)
        ...
        devise_scope mapping.name do
          with_devise_exclusive_scope mapping.fullpath, mapping.name, options do
            routes.each { |mod| send("devise_#{mod}", mapping, mapping.controllers) }
          end
        end
      end
~~~

同时，Mapping的initialize方法定义了成员变量：scope以及concern和nesting。这些变量存储了scope，concern以及nesting的一些配置。

~~~rb
      def initialize(set) #:nodoc:
        @set = set
        @scope = { :path_names => @set.resources_path_names }
        @concerns = {}
        @nesting = []
      end
~~~


其根据你传入的hash参数进行一系列组装（默认的值的加入）后，其最终会调用add_route方法。

~~~rb
def add_route(action, options) # :nodoc:
  path = path_for_action(action, options.delete(:path))
  action = action.to_s.dup

  if action =~ /^[\w\-\/]+$/
    options[:action] ||= action.tr('-', '_') unless action.include?("/")
  else
    action = nil
  end

  if !options.fetch(:as, true)
    options.delete(:as)
  else
    options[:as] = name_for_action(options[:as], action)
  end

  mapping = Mapping.new(@set, @scope, URI.parser.escape(path), options)
  app, conditions, requirements, defaults, as, anchor = mapping.to_route
  @set.add_route(app, conditions, requirements, defaults, as, anchor)
end
~~~

根据你传入的参数Mapping，比如sprocket-rails生成的mapping长这样

~~~rb
=> #<ActionDispatch::Routing::Mapper::Mapping:0x007fa216bd0820
 @conditions={:path_info=>"/assets", :required_defaults=>[]},
 @constraints={},
 @defaults={},
 @options=
  {:to=>
    #<Sprockets::Environment:0x3fd10a2cd948 root="/Users/soulspirit/GitHub/ninja", paths=["/Users/soulspirit/ninja/app/assets/images", "/Users/soulspirit/ninja/app/assets/javascripts", "/Users/soulspirit/ninja/app/assets/stylesheets",...>,
   :anchor=>false,
   :format=>false},
 @path="/assets",
 @requirements={},
 @scope={:path_names=>{:new=>"new", :edit=>"edit"}},
 @segment_keys=[],
 @set=#<ActionDispatch::Routing::RouteSet:0x007fa215cfb638>>
~~~

总结：

1 一个Rails应用中会有一个RouteSet，可以通过Rails.application.routes访问。
2 RouteSet中有一个Routes,我们可以通过Rails.application.routes.routes或者Rails.application.routes.set来访问，它保存了Rails中的routes。
3 Routes中有一个routes数组，这其中就是保存的routes。
4 RouteSet和Routers中各维护了一个named_routes，这是为毛？
5 RouteSet中包含router,routes其中router是包含了routes。
Rails.application.routes.router.routes == Rails.application.routes.routes
6 一个Route的模样长这样,它包含一个Dispatcher。

~~~rb
[18] pry(#<ActionDispatch::Journey::Routes>)> route
=> #<ActionDispatch::Journey::Route:0x007fbc1da52810
 @app=
  #<ActionDispatch::Routing::RouteSet::Dispatcher:0x007fbc1b67ba90
   @controller_class_names=#<ThreadSafe::Cache:0x007fbc1b67b9a0 @backend={}, @default_proc=nil>,
   @defaults={:controller=>"home", :action=>"index"},
   @glob_param=nil>,
 @constraints={:required_defaults=>[:controller, :action], :request_method=>/^GET$/},
 @decorated_ast=nil,
 @defaults={:controller=>"home", :action=>"index"},
 @dispatcher=true,
 @name="root",
 @parts=nil,
 @path=
  #<ActionDispatch::Journey::Path::Pattern:0x007fbc1db4a2e0
   @anchored=true,
   @names=[],
   @offsets=nil,
   @optional_names=nil,
   @re=nil,
   @required_names=nil,
   @requirements={},
   @separators="/.?",
   @spec=#<ActionDispatch::Journey::Nodes::Slash:0x007fbc1db491d8 @left="/", @memo=nil>>,
 @precedence=0,
 @required_defaults=nil,
 @required_parts=nil>
~~~

7 Router用于执行请求，ast是拿来干嘛的,simulator是拿来干嘛的。

## Routes的匹配

1. route有优先级，就是Route中的precedence字段。
2.

在上节中提到request经过一层层的middleware最后会走到router的call方法中。

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

其中find_routes(env)就是Rails根据路径寻找路由的方法。find_routes会根据GTG和NFA算法来匹配路由，这个以后研究。在这里，find_routes会返回一个路由数组，然后通过调用route中的app的call方法来得到结果。

~~~rb
def call(env)
  params = env[PARAMETERS_KEY]

  # If any of the path parameters has an invalid encoding then
  # raise since it's likely to trigger errors further on.
  params.each do |key, value|
    next unless value.respond_to?(:valid_encoding?)

    unless value.valid_encoding?
      raise ActionController::BadRequest, "Invalid parameter: #{key} => #{value}"
    end
  end

  prepare_params!(params)

  # Just raise undefined constant errors if a controller was specified as default.
  unless controller = controller(params, @defaults.key?(:controller))
    return [404, {'X-Cascade' => 'pass'}, []]
  end

  dispatch(controller, params[:action], env)
end

private
def dispatch(controller, action, env)
  controller.action(action).call(env)
end
~~~

