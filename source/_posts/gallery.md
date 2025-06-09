---
title: 简单易懂的本地云图库搭建
date: 2025-06-09 20:06:05
tags:
  - 图库
categories:
  - [ 运维技术, 私有云 ]
top_img: /img/top.png
cover: /img/yw.png
---

### 环境生态
由于本篇是基于docker生态搭建，需要用户对docker生态有一点运用知识。

### docker-compose一键化安装
用文档新建`docker-compose.yml`文件，将以下配置复制进去。
```
services:
  mysql:
    image: mysql:latest
    container_name: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - <自定义数据挂载地址，如：E:>/mysql/data/:/var/lib/mysql
    ports:
      - "3306:3306"
      - "33060:33060"
    restart: unless-stopped
  piwigo:
    image: linuxserver/piwigo:latest # 或者你构建的镜像名称
    container_name: piwigo
    ports:
      - "28081:80"
    volumes:
      - <自定义图库挂载地址>/gallery:/gallery
      - <自定义图库挂载地址>/piwigo/config:/config
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    restart: unless-stopped
    depends_on:
      - mysql
```
在文件同级目录下执行命令行`docker-compose up -d`docker将会自动安装并执行.等待5-10分钟.
成功后执行`docker ps`查看容器是否正常运行。
![docker1.jpg](docker1.jpg)

### 配置安装
成功后打开localhost:28081,可以看见初始化界面。这里注意先在mysql中建立数据库`piwigo`。最后点击安装等待5分钟打开 http://localhost:28081/。
![piwigo.jpg](piwigo.jpg)
> 注意：如果是按上面的步骤使用的同一docker机中的mysql容器.主机地址不能设置`127.0.0.1`或`localhost`.必须设置`host.docker.internal`.


### 本地图片文件夹迁移进图库
#### 将本地图片文件夹复制到`docker-compose.yml`中的`<自定义图库挂载地址>/galleries`路径下。注意文件夹和图片文件皆不能中文命名。
![j1.png](j1.png)

#### 从图库进入控制台同步本地图片文件夹
![piwigo-consol.jpg](piw-consol.jpg)
![img.png](piw-admin.png)

#### 修改图库名称
![j2.png](j2.png)

#### 最终展现
![j3.png](j3.png)
> 可以通过`控制台->设置->主题`修改整体主题。

#### 在个人博客中的应用
![j4.png](j4.png)