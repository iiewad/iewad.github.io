---
layout: post
title: ActiveAdmin 数据图表
date: 2018-10-26 10:25:35 +0800
categories: nginx
---

### Gemfile
Gemlist:
- [groupdate](https://github.com/ankane/groupdate)
- [chartkick](https://github.com/ankane/chartkick)
```ruby
#Gemfile
gem 'groupdate'
gem "chartkick"
```

执行`$ bundle install`

### ActiveAdmin 引入图表js库

```ruby
#config/initializers/active_admin.rb
ActiveAdmin.setup do |config|
  config.register_javascript 'https://www.gstatic.com/charts/loader.js'
end
```

```coffee
# app/assets/javascripts/active_admin.js.coffee
#= require active_admin/base
#= require chartkick
```

### ActiveAdmin 添加图表页面

```shell
$ rails g active_admin:page StuUserChart
Running via Spring preloader in process 87926
      create  app/admin/stu_user_chart.rb

```

```
#  app/admin/stu_user_chart.rb
ActiveAdmin.register_page "StuUserChart" do
  content do
    column_chart [["2016-01-01", 30], ["2016-02-01", 54]], stacked: true, library: {colors: ["#D80A5B", "#21C8A9", "#F39C12", "#A4C400"]}
    #para line_chart StuUser.group_by_day(:created_at).count
  end
end
```

### 有帮助的链接
- [Adding Google Charts to your Active Admin Application](https://spin.atomicobject.com/2016/11/23/adding-google-charts-active-admin-application/)
- [Custom activeadmin pages with charts](http://juanda.me/create-custom-activeadmin-pages-with-charts)
