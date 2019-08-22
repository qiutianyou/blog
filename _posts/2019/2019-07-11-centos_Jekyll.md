---
layout: post
title: centOS安装Jekyll
category: Jekyll
tags: [Linux]
---

启动我的博客项目，因为使用的Jekyll模版，所以需要需要安装并启动服务

官网：[https://jekyllrb.com/](https://jekyllrb.com/)
给出了几条安装和启动服务的命令

```
gem install bundler jekyll

jekyll new my-awesome-site

cd my-awesome-site

bundle exec jekyll serve
```

默认centos是没有安装gem的

```
[qiu@localhost Desktop]$ gem install bundler jekyll
bash: gem: 未找到命令...
```

## 安装ruby


### 安装GPG密钥
```
$ gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```

### 安装RVM（ruby版本管理）

```
$ \curl -sSL https://get.rvm.io | bash -s stable
```

提示“Thanks for installing RVM”，则安装成功了。
参考：[http://rvm.io/](http://rvm.io/)



刷新RVM,/home/qiu/ 是用户目录

```

$ source /home/qiu/.rvm/scripts/rvm
```




### 安装指定版本的ruby，这里我安装的是2.3.7.因为我mac本地安装的就是这个版本

安装时间会比较长，莫着急

```
$ rvm install 2.3.7
```

### 如果rvm install安装报错

```
Error running 'requirements_centos_libs_install autoconf automake bison libffi-devel libtool readline-devel ruby sqlite-devel openssl-devel',
please read /home/qiu/.rvm/log/1563033625_ruby-2.3.7/package_install_autoconf_automake_bison_libffi-devel_libtool_readline-devel_ruby_sqlite-devel_openssl-devel.log
Requirements installation failed with status: 1.
```

安装yaml

```
$ sudo yum install libyaml
```

重新安装

```
$ rvm install 2.3.7
```

### 安装完成
rvm安装ruby的时候，会有提示log，提示安装在哪里。"/home/qiu/.rvm/src/ruby-2.3.7" 便是安装的路径


```
ruby-2.3.7 - #extracting ruby-2.3.7 to /home/qiu/.rvm/src/ruby-2.3.7.....
ruby-2.3.7 - #configuring...........................................................

```

完成提示"Install of ruby-2.3.7 - #complete"

#### 检查版本

```
[root@localhost ruby]# gem -v
2.0.14.1
[root@localhost ruby]# ruby -v
ruby 2.0.0p648 (2015-12-16) [x86_64-linux]

```

使用rvm切换ruby版本

```
[qiu@localhost ~]$ rvm use 2.3.7 --default
```



## 安装jekyll

若提示版本太低，“public_suffix requires Ruby version >= 2.1.”。要么重启试试

```
sudo gem install yekyll
sudo gem install bundler jekyll
```


## 启动服务
```
$ git clone https://github.com/qiutianyou/qiutianyou.github.io.git
$ cd qiutianyou.github.io
$ sudo bundle install
$ bundle exec jekyll serve
```

提示“Server address: http://127.0.0.1:4000/”，则服务启动成功，访问对应地址
