---
layout: post
title: nginx 反向代理Apache
date: 2018-08-08 17:42:03 +0800
categories: nginx
---

## 废话
我的Wordpress原本是丢在腾讯云低配（1+1+20）的学生机上，现在毕业不再继续使用。
![image](https://note.youdao.com/favicon.ico)
服务器运行着Rails App，因为各种环境搭起来心太累，所以我直接使用 [LAMP](https://lnmp.org/)
```shell
wget http://soft.vpser.net/lnmp/lnmp1.5.tar.gz -cO lnmp1.5.tar.gz && tar zxf lnmp1.5.tar.gz && cd lnmp1.5 && ./install.sh lamp
```
这里注意使用`sudo`

经过不知道几分钟的等待就能搭好，基本不会出问题。

## 先把phpmyadmin跑起来
Lamp环境默认www路径```/home/wwwroot/default/```

phpmyadmin路径```/home/wwwroot/default/phpmyadmin/```

1. 编辑Apache配置文件
    ```shell
    # Listen 80
    Listen 8080
    ```
2. 添加VHost
    ```shell
    deployer@VM-0-14-ubuntu:~$ sudo lnmp vhost add
    +-------------------------------------------+
    |    Manager for LNMP, Written by Licess    |
    +-------------------------------------------+
    |              https://lnmp.org             |
    +-------------------------------------------+
    Please enter domain(example: www.lnmp.org): phpmyadmin.domain.com
     Your domain: phpmyadmin.domain.com
    Enter more domain name(example: lnmp.org *.lnmp.org):
    Please enter the directory for the domain: phpmyadmin.domain.com
    Default directory: /home/wwwroot/phpmyadmin.domain.com:
    Virtual Host Directory: /home/wwwroot/phpmyadmin.domain.com
    Allow access log? (y/n)
    Disable access log.
    Please enter Administrator Email Address: admin@domain.com
    Server Administrator Email:admin@domain.com
    Create database and MySQL user with same name (y/n) n
    Add SSL Certificate (y/n) n
    
    Press any key to start create virtul host...
    
    Create Virtul Host directory......
    set permissions of Virtual Host directory......
    Test Apache configure file...
    test apache configure... Syntax OK
     done
    Restart Apache...
    restart apache...  done
    ================================================
    Virtualhost infomation:
    Your domain: phpmyadmin.domain.com
    Home Directory: /home/wwwroot/phpmyadmin.domain.com
    Enable log: yes
    Create database: no
    Create ftp account: no
    ================================================
    ```

3. 因为这里创建了`/home/wwwroot/phpmyadmin.domain.com`这个路径，等下要给phpmyadmin建立软链接，我先给它删掉

    ```shell
    deployer@VM-0-14-ubuntu:/home/wwwroot$ sudo rm -rf phpmyadmin.domain.com/
    deployer@VM-0-14-ubuntu:/home/wwwroot$ ls
    default  lidawei.me  phpiewadmin.lidawei.me  tmp
    deployer@VM-0-14-ubuntu:/home/wwwroot$ clear
    deployer@VM-0-14-ubuntu:/home/wwwroot$ ls
    default  lidawei.me  phpiewadmin.lidawei.me  tmp
    deployer@VM-0-14-ubuntu:/home/wwwroot$ sudo ln -s /home/wwwroot/default/phpmyadmin/ /home/wwwroot/phpmyadmin.domain.com
    deployer@VM-0-14-ubuntu:/home/wwwroot$ ll
    总用量 20
    drwxr-xr-x 5 root root 4096 8月   8 18:01 ./
    drwxr-xr-x 7 root root 4096 8月   8 14:55 ../
    drwxr-xr-x 3 www  www  4096 8月   7 10:20 default/
    drwxr-xr-x 6 www  www  4096 8月   8 17:02 lidawei.me/
    lrwxrwxrwx 1 www  www    32 8月   8 15:11 phpiewadmin.lidawei.me -> /home/wwwroot/default/phpmyadmin/
    lrwxrwxrwx 1 root root   33 8月   8 18:01 phpmyadmin.domain.com -> /home/wwwroot/default/phpmyadmin//
    drwxr-xr-x 3 root root 4096 8月   5 23:44 tmp/
    deployer@VM-0-14-ubuntu:/home/wwwroot$
    ```
    
4. 编辑VHost配置文件端口改为8080
    ```shell
    deployer@VM-0-14-ubuntu:~$ vi /usr/local/apache/conf/vhost/phpmyadmin.domain.com.conf
    <VirtualHost *:8080>
    ServerAdmin admin@domain.com
    php_admin_value open_basedir "/home/wwwroot/phpmyadmin.domain.com:/tmp/:/var/tmp/:/proc/"
    DocumentRoot "/home/wwwroot/phpmyadmin.domain.com"
    ServerName phpmyadmin.domain.com
    #ErrorLog "/home/wwwlogs/phpmyadmin.domain.com-error_log"
    #CustomLog "/home/wwwlogs/phpmyadmin.domain.com-access_log" combined
    <Directory "/home/wwwroot/phpmyadmin.domain.com">
        SetOutputFilter DEFLATE
        Options FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        DirectoryIndex index.html index.php
    </Directory>
    </VirtualHost>
    ```
    
 
 5. 添加nginx的配置文件
    ```shell
    deployer@VM-0-14-ubuntu:/etc/nginx/conf.d$ sudo cp default.conf.bak phpmyadmin.domain.conf
    deployer@VM-0-14-ubuntu:/etc/nginx/conf.d$ sudo vi phpmyadmin.domain.conf
    server {
    listen       80;
    server_name  phpmyadmin.domain.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    #location ~ \.php$ {
    #    proxy_pass    http://127.0.0.1:8080
    #}


    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
    }
    ```
    
6. 重启nginx和Apache
    ```shell
    deployer@VM-0-14-ubuntu:/etc/nginx/conf.d$ sudo nginx -t
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    deployer@VM-0-14-ubuntu:/etc/nginx/conf.d$ sudo nginx -s reload
    deployer@VM-0-14-ubuntu:/etc/nginx/conf.d$ sudo lnmp restart
    +-------------------------------------------+
    |    Manager for LNMP, Written by Licess    |
    +-------------------------------------------+
    |              https://lnmp.org             |
    +-------------------------------------------+
    Stoping LAMP...
    stop apache...  done
    [ ok ] Stopping mysql (via systemctl): mysql.service.
    Starting LAMP...
    start apache...  done
    [ ok ] Starting mysql (via systemctl): mysql.service.
    ```
    

7. 域名解析不解释了
    