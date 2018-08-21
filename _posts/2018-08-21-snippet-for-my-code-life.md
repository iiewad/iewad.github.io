---
layout: post
title:  "开发过程的一些记录"
date:   2018-08-21 16:00:00 +0800
categories: blog
tag: ruby, linux
---

### 1. 查看端口被哪个程序占用
`sudo lsof -i tcp:port`

### 2. 看到进程的PID，可以将进程杀死。
`sudo kill -9 PID`

### 文件查找
`locate nginx.conf`

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
