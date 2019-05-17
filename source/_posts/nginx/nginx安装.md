---
title: nginx安装
date: 2018-06-11 20:12:04
tags: [nginx]
categories: nginx
---

## 官网下载 nginx

  - 1 下载 Nginx，下载地址：http://nginx.org/download/nginx-1.6.2.tar.gz,并上传到服务器，或直接使用wget 

## 安装 nginx

  - 1 解压安装包 
     ```
       tar zxvf nginx-1.6.2.tar.gz
     ```
  
  - 2 进入解压目录
     ```
       cd nginx-1.6.2
     ```
  - 3 编译安装
       ```
         命令：
              ./configure --prefix=/usr/local/nginx
               make && make install
         错误：./configure: error: the HTTP rewrite module requires the PCRE library.如果出现以上错误
               解决如下：
                       yum -y install pcre-devel openssl openssl-devel
                       然后重新执行上一步命令
       ```
## 安装完成

   - 1 查看版本
        ```
          /usr/local/nginx/sbin/nginx -v
        ```
   - 2 启动Nginx
        ```
          /usr/local//nginx/sbin/nginx
        ```
   - 3 Nginx其他命令
        ```
          /usr/local/nginx/sbin/nginx -s reload            # 重新载入配置文件
          /usr/local/nginx/sbin/nginx -s reopen            # 重启 Nginx
          /usr/local/nginx/sbin/nginx -s stop              # 停止 Nginx
        ```  
## 访问Nginx
    
   - 1 阿里云服务器默认80端口关闭，需要在控制台管理界面，左侧菜单 安全组-->配置规则，添加80/80端口。即可访问