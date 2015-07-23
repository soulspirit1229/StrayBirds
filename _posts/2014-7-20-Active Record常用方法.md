---
layout: post
title: Active Record常用方法
category: 总结
comments: true
---

# Active Record

<!-- create time: 2014-11-20 21:50:10  -->

## find
    Task.find(:all, :conditions => ["complete=? and priority=?", false, 3])
    Task.find(:all, :conditions => ["complete=? and priority IS ?", false, nil])
    Task.find(:all, :conditions => ["complete=? and priority IN (?)", false, [1,3]])
    Task.find(:all, :conditions => ["complete=? and priority IN (?)", false, 1..3])
    Task.find(:all, :conditions => { :complete => false, :priority => 1 })
    Task.find(:all, :conditions => { :complete => false, :priority => nil })
    Task.find(:all, :conditions => { :complete => false, :priority => [1,3] })
    Task.find(:all, :conditions => { :complete => false, :priority => 1..3 })
    Task.find_all_by_priority(1..3)
    Task.find_by_complete(false, :order => 'created_at DESC')



##### move find method to model, not in controller

## scope
    scope :published, -> { where(published: true) }
    scope :published_and_commented, -> { published.where("comments_count > 0") }
    scope :created_before, ->(time) { where("created_at < ?", time) } ＃传参数
    default_scope { where state: 'pending' } # default scope跟其它scope是and的关系
    Client.unscoped.load ＃去除所有scope，正常的query

## eager loading
    Task.find(:all, :include => :projects)
    Task.find(:all, :include => [:projects, :comments])
    Task.find(:all, :include => [:projects, {:comments => :user}])
    Category.includes(posts: [{ comments: :guest }, :tags]).find(1)

## SQL injection
    @tasks = Task.find(:all, :conditions => ["name LIKE ?", "%#{params[:query]}%"])

