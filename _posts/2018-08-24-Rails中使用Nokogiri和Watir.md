---
layout: post
title: Rails中使用Nokogiri和Watir
date: 2018-08-24 09:24:12 +0800
categories: nginx
---

最近我的Rails App. 中，有场景需要抓取学校官网某栏目的新闻内容为小程序提供数据，故使用了`Nokogiri`来实现，因该页面的分页是用Js来渲染的导致在Nokogiri查找分页节点的时候出现异常，后经搜索得知有`Watir`这个Gem。

附分页渲染代码：
```erb
<SCRIPT>
//createPageHTML(9, 0, "index", "html");
var currentPage = 0;//所在页从0开始
//var headPage = "index"+"."+"html";//首页
//var tailPage = "index_" + (countPage-1) + ".html"//尾页
var prevPage = currentPage-1;//上一页
var nextPage = currentPage+1;//下一页
var countPage = 17;//共多少页


//设置上一页代码
if(countPage>1&&currentPage!=0&&currentPage!=1)
document.write("<td><a href=\"index.html\">首页</a></td><td><a class='prev' href=\"index"+"_" + prevPage + "."+"html\">上一页</a></td>");
else if(countPage>1&&currentPage!=0&&currentPage==1)
document.write("<td><a href=\"index.html\">首页</a></td><td><a class='prev' href=\"index.html\">上一页</a></td>");
else
document.write("<td><a class='prev'>首页</a></td><td><a class='pre'>上一页</a></td>");
//循环
var num = 8;
for(var i=0+(currentPage-1-(currentPage-1)%num) ; i<=(num+(currentPage-1-(currentPage-1)%num))&&(i<countPage) ; i++){
if(currentPage==i)
document.write("<td><a class='curr'>"+(i+1)+"</a></td>");
else if(i==0){
document.write("<td><a class='num'href=\"index"+"."+"html\">"+1+"</a></td>");}
else
document.write("<td><a class='num' href=\"index"+"_" + i + "."+"html\">"+(i+1)+"</a></td>");
}
//设置下一页代码 
if(countPage>1&&currentPage!=(countPage-1))
document.write("<td><a  class='next' href=\"index"+"_" + nextPage + "."+"html\">下一页</a></td><td><a  class='prev' href=\"index_" + (countPage-1) + ".html\">末页</a></td>");
else
document.write("<td><a class='next'>上一页</a></td><td><a class='prev'>末页</a></td>");


function toPage(){
var _num = document.getElementById("num").value;
var str = "index"+"_"+(_num-1)+"."+"html";
var url = location.href.substring(0,location.href.lastIndexOf("/")+1);
if(_num<=1||_num==null)
location.href = url+"index"+"."+"html";
else if(_num>countPage)
alert("本频道最多"+countPage+"页");
else
location.href = url+str;
} 
</SCRIPT>
```

# 安装
```ruby
#Gemfile
gem 'watir', '~> 6.12'
```
`bundle install` 总是不难的

接下来在Spider Task中使用
```ruby
namespace :spiders do
  require 'watir'

  desc 'Spider => describe'
  task get_xshd: :environment do
    @start_url = "http://example.edu.cn"
    get_lists(@start_url, 'get_all')
  end


  def get_lists(url, mode = 'update')
    browser = Watir::Browser.new :chrome, headless: true
    browser.goto url
    #browser = Watir::Browser.start url
    doc = Nokogiri::HTML.parse(browser.html)
    browser.close
    ·······
  end
```

可以看到，我开始是直接
`browser = Watir::Browser.start url` 但是这样在我的服务器上出现了异常，经过折腾改变了写法，异常如下：

```ruby
Selenium::WebDriver::Error::UnknownError: unknown error: Chrome failed to start: exited abnormally
  (unknown error: DevToolsActivePort file doesn't exist)
  (The process started from chrome location /usr/bin/google-chrome is no longer running, so ChromeDriver is assuming that Chrome has crashed.)
  (Driver info: chromedriver=2.41.578700 (2f1ed5f9343c13f73144538f15c00b370eda6706),platform=Linux 4.4.0-91-generic x86_64)
```

我感觉是因为`new` 和 `start`的区别，我没有去验证这个区别。

至此已经可以正常使用了。

---

# 一些需要注意的
- `Watir` 是会用到browserdriver的，所以务必安装好**browserdriver**，我这里使用的是`chromedriver`
- 服务器上安装google-chrome

```shell
    sudo curl -sS -o - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add
    sudo echo "deb [arch=amd64]  http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list
    sudo apt-get -y update
    sudo apt-get -y install google-chrome-stable
```

- 安装chromedriver(这里要根据自己的环境做适当修改)
    
```shell
    wget https://chromedriver.storage.googleapis.com/2.41/chromedriver_linux64.zip
    unzip chromedriver_linux64.zip
    mv chromedriver /usr/bin/
    ### 或者
    sudo apt-get install chromium-chromedriver
```

    
# End