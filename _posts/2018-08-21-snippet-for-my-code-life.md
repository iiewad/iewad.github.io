---
layout: post
title:  "开发过程的一些记录"
date:   2018-08-21 16:00:00 +0800
categories: blog
tag: ruby, linux
---

### 查看端口被哪个程序占用
`sudo lsof -i tcp:port`

### 看到进程的PID，可以将进程杀死。
`sudo kill -9 PID`

### 文件查找
```
locate nginx.conf
find . -name game
```

### 利用pwgen生成密码
`pwgen`

### 命令行插入代码至文件(mac)
```shell
-> sed -i '' '1i\
quote> hello
quote> ' hello.txt
```

### ruby解析xml节点
```ruby
require 'rexml/document'
include REXML
xmlstr = '<?xml version="1.0" encoding="utf-8"?>
<string xmlns="MUI">{"rList":[{"DormId":"G1276242854","DormName":"东湖公寓","RoomId":"","RoomAccountID":""},{"DormId":"G1290492858","DormName":"丰泽公寓","RoomId":"","RoomAccountID":""},{"DormId":"G1283531903","DormName":"金岸公寓","RoomId":"","RoomAccountID":""},{"DormId":"G1252577939","DormName":"芷兰公寓","RoomId":"","RoomAccountID":""}],"Status":"1","Message":"请求成功","rowCount":4}</string>'

xmldoc = Document.new(xmlstr)
 
# 获取 root 元素
root = xmldoc.root

result = JSON.parse(root.get_text.to_s)

```

### ruby将Hash的key转为小写
```ruby
hash.transform_keys!(&:downcase)
```

### ls显示文件大小
```shell
ls -lh ~
```

### scp 文件上传/下载
```shell
# upload
scp -P 8864 ./tmp/member.xls deployer@test-17pdf.kdan.cn:/var/www

# download
scp -P 2222 root@www.vpser.net:/root/lnmp0.4.tar.gz /home/lnmp0.4.tar.gz
```

### wordpress .htaccess
```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
```

### VIM 强写
```vim
:w !sudo tee %
```

### Rails4+ 不验证authenticity_token
```ruby
skip_before_action :verify_authenticity_token
```

### GitHub 空项目
```
# create a new repository on the command line
echo "# golang-book-practice" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:iiewad/golang-book-practice.git
git push -u origin master
```
```
#push an existing repository from the command line
git remote add origin git@github.com:iiewad/golang-book-practice.git
git push -u origin master
```
[iewad](https://iewad.me/)

### Terminal 代理
```shell
export http_proxy='http://localhost:8118'
export https_proxy='http://localhost:8118'

#取消代理

unset http_proxy
unset https_proxy
```

### tmux备忘
```
tmux new -s session_name
tmux a -t session_name
tmux switch -t session_name

```

### rvm gem install
```
rvm @global do gem install bundle
rvm rubygems current #update gems
```
```
gem install libv8 -v '3.16.14.13'  -- --with-system-v8
```

### css文本溢出处理
```css
单行：
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;

多行：
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```

### tar 解压缩
```
压缩
tar -czvf name-of-archive.tar.gz /path/to/directory-or-file
#解压
tar -xvzf name-of-archive.tar.gz
```

### Postgres导入导出
postgresql 导出的数据库， 标准语句，

pg_dump --host [**地址**] --port [**端口**] --username [**数据库的用户名**] > [**导出的文件**] [**数据库名字**]

例子：
```
pg_dump --host xxxxx.com --port 3434 --username cs  > cs.sql cs
```

postgresql导入数据库， 标准语句：

psql -d [**数据库名字**] -f [**文件名**] [**用户名**]

例子：
```
psql -d cs  -f cs.sql cs
```
