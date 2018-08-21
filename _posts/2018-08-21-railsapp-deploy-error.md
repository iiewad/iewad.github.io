---
layout: post
title:  "Rails App. 部署出错排障"
date:   2018-08-21 11:33:59 +0800
categories: rails
tag: ruby, rails
---

## 错误信息：

```shell
deployer@test-center:/var/www/my_vpn_api_server/current$ RAILS_ENV=staging rails assets:precompile
rails aborted!
ExecJS::ProgramError: TypeError: Object #<Object> has no method 'log2'
unpackSupport ((execjs):14780:36)
(execjs):14794:28
Array.reduce (native)
(execjs):14792:70
Array.reduce (native)
unpackFeature ((execjs):14790:44)
f ((execjs):20:10)
Object.caniuse-lite ((execjs):88:1)
o ((execjs):1:913)
(execjs):1:964
```

## 描述：
一个很简单的新项目，在测试机执行以上命令对前端资源进行预编译时发生了意想不到的错误，在通过搜索和请教Leader之后，得到了以下建议：
- 加 bundle exec
- 安装 nodejs

## 折腾

### 尝试用最简单的方法
```shell
RAILS_ENV=staging bundle exec rake assets:precompile
```
偷懒还是有点问题的，依旧是同样的报错

### 锁定错误原因

根据网络搜索和Leader建议再加上以上尝试，断定出错原因是staging环境node环境异常了（Production环境并未出现错误）

当我执行`node --version` || `nodejs --version` 时候看到node版本还是0.10.x

所以果断重新安装node

`sudo apt-get purge nodejs`

`sudo apt-get install nodejs`

.....

再查看node版本，神奇的一幕出现了
`nodejs -v` || `node -v` 版本不一样 =。-

想办法全局卸载了所有的node || nodejs 版本
```shell
sudo apt-get remove nodejs

sudo apt-get remove npm

sudo rm -rf /usr/local/bin/npm /usr/local/share/man/man1/node* /usr/local/lib/dtrace/node.d ~/.npm ~/.node-gyp /opt/local/bin/node /opt/local/include/node /opt/local/lib/node_modules 

sudo rm -rf /usr/local/lib/node*

sudo rm -rf /usr/local/include/node*

sudo rm -rf /usr/local/bin/node*

```

#### 使用nvm 安装node
```shell
sudo apt-get update
sudo apt-get install build-essential libssl-dev
curl -sL https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh -o install_nvm.sh
bash install_nvm.sh
source ~/.profile

nvm ls-remote
...

nvm install 10.8.0

nvm use 10.8.0

node -v
v10.4.0
```

oj8k 这下该解决了

```shell
cd /path/project/current
RAILS_ENV=staging bundle exec rake assets:precompile
...
...
...
```

正常编译了

### But

部署：
```shell
cap staging deploy

Gem Load Error is: Could not find a JavaScript runtime. See https://github.com/rails/execjs for a list of available runtimes.
```

根据错误原因进行搜索：
> execjs会选择一个js解释器， therubyracer 或者 nodejs选择一个就行了 ruberacer需要在Gemfile里指定，如果系统有nodejs，那么不需要在Gemfile里指定了 --from ruby-china

看上面搞了半天还是node没安装好的缘故，只好继续去服务器排障：
查看了node的版本是正常的，nodejs好像并不是很正常，执行`nodejs --version`时，提示nodejs未安装。

搞了这几波node有没有安装心里还是有点数的。搜索
> node && nodejs different version

```shell
$ sudo apt-get remove nodejs
$ sudo ln -s /usr/bin/node /usr/bin/nodejs
```
这里建立软连接的时候要注意自己的node路径。
我用nvm安装的node路径是
```shell
deployer@test-center:~$ which node
/home/deployer/.nvm/versions/node/v10.8.0/bin/node
```
所以我建立软链的命令应该是

`sudo ln -s /home/deployer/.nvm/versions/node/v10.8.0/bin/node /usr/bin/nodejs`

最后，确实是成功了。
