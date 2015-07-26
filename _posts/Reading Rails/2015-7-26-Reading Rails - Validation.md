---
layout: post
title: Reading Rails - Rails Validation
category: Reading Rails
comments: true
---

# Reading Rails - Validation
在Rails中，有两个模块具有validation，一个是ActiveModel::Valdiations,另一个是ActiveRecord::Validations
.

比如我们常见的validates来自Model，而valid?来自ActiveRecord。

## ActiveModel::Validations
我们先来看下ActiveModel::Validations.

~~~rb
[6] pry(main)> class MyValidation
[6] pry(main)*   include ActiveModel::Validations
[6] pry(main)* end
=> MyValidation
[7] pry(main)> MyValidation.ancestors
=> [MyValidation,
 ActiveModel::Validations::HelperMethods,
 ActiveSupport::Callbacks,
 ActiveModel::Validations,
 Object,
 PP::ObjectMixin,
 Delayed::MessageSending,
 ActiveSupport::Dependencies::Loadable,
 JSON::Ext::Generator::GeneratorMethods::Object,
 Kernel,
 BasicObject]
~~~
