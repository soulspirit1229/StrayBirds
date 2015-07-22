---
layout: post
title: Tips - FactoryGirl的总结帖
category: Tips
comments: true
---

### Factory Girl

#### Best Practice
* 基本的factory中生产的model只需要是满足require的attribute就可以。
If you're creating ActiveRecord objects, that means that you should only provide attributes that are required through validations and that do not have defaults. Other factories can be created through inheritance to cover common scenarios for each class.

#### synax
##### sequence

    expect(actual).to match(/expression/)

***
##### association
  association :contact

***
##### build
  FactoryGirl.build(:contact)

***
##### attributes_for
  FactoryGirl.attributes_for(:contact) # generate hash value based on the object

***
##### config
spec/spec_helper.rb

* config.include FactoryGirl::Syntax::Methods #Include Factory Girl syntax to simplify calls to factories
* config.include LoginMacros #include the module that we created

***
#### [callback](http://robots.thoughtbot.com/get-your-callbacks-on-with-factory-girl-3-3)
* after(:build)
* before(:build)
* after(:create)
* before(:build)
