---
title: 通过Apache部署gitbook静态网页
date: 2020-08-11 11:03:07
tags: web
category: gitbook
---

### 通过apache部署gitbook

###### 之前搭建的gitbook server需要通过端口号来访问，这样很丑也不安全，今天闲来无事（上班划水）就把它放在了apache上面。我这里用的是ubuntu的sever，在centos这样的red hat系的系统里可能会有一些差别。

##### 1.安装apache

red hat系的linux系统一般自带apache不需要自行安装，ubuntu在官方教程中有apache的安装和部署样例：[官方样例](https://ubuntu.com/tutorials/install-and-configure-apache#1-overview) 。

```bash
sudo apt update
sudo apt install apache2
```

在安装完成后，如果服务器开放了80端口，就可以通过域名或者ip访问到部署好的如下图测试网页。

![apache defult page](https://ubuntucommunity.s3.dualstack.us-east-2.amazonaws.com/original/2X/7/771159b35c97e429247aac754ad44bf06cc1efa8.png)

##### 2. 生成网站目录

在ubuntu中apache的默认静态网页文件在`/var/www/html `中。推荐的方式是将你的静态网页放在`/var/www/`下。所以我们在这个目录下生成一个文件夹作为存放gitbook的html文件的位置。

```bash
sudo mkdir /var/www/gitbook
```

##### 3.build gitbook文件夹下的md文件生成静态网页存在`/var/www/gitbook`下

```bash
gitbook build [your gitbook path] --output= /var/www/gitbook
```

##### 4. 修改apache配置文件

所有的配置文件都被存在`/etc/apache2/site-avaliable/`下。

在该目录下生成自定义的配置文件`gitbook.conf`并且编辑：

```
DocumentRoot /var/www/gci/
```

##### 5.重启apache服务

首先将配置文件改为我们自定义的配置文件

```bash
sudo a2ensite gitbook.conf
```

然后重启apache

```bash
service apache2 reload
```

##### 这样我们就可以通过访问自己的域名来访问自己的gitbook啦

![](https://res.cloudinary.com/afan1996/image/upload/v1597143635/poop_vgin0f.png)


