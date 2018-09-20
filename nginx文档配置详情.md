## Nginx配置统计流量带宽请求及记录实时请求状态的方法


### 这篇文章主要介绍了Nginx中配置统计流量带宽请求及记录实时请求状态的方法,分别用到了ngx_req_status和ngx_realtime_request模块,需要的朋友可以参考下

#### 流量带宽请求状态统计
ngx_req_status用来展示nginx请求状态信息，类似于apache的status，nginx自带的模块只能显示连接数等等信息，我们并不能知道到底有哪些请求、以及各url域名所消耗的带宽是多少。ngx_req_status提供了

- 这些功能.
- 功能特性
- 按域名、url、ip等等统计信息
- 统计总流量
- 统计当前带宽\峰值带宽
- 统计总请求数量


## 1. 安装

添加nginx的yum源:


    sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm


#

     # cd /usr/local/src/
     # yum -y install pcre-devel openssl openssl-devel gcc
     # wget "http://nginx.org/download/nginx-1.7.4.tar.gz  "
     # tar -xzvf nginx-1.7.4.tar.gz
     # wget https://github.com/zls0424/ngx_req_status/archive/master.zip -O ngx_req_status.zip
     # unzip ngx_req_status.zip
     # cd nginx-1.7.4/
     # patch -p1 < ../ngx_req_status-master/write_filter.patch
     # ./configure --prefix=/usr/local/src/nginx --add-module=../ngx_req_status-master
     # make -j2
     # make install


## 2. 配置

    http {
     req_status_zone server_name $server_name 256k;
     req_status_zone server_addr $server_addr 256k;
     req_status_zone server_url $server_name$uri 256k;
     req_status server_name server_addr server_url;
    server {
     server_name 127.0.0.1;
     location /ttlsa-req-status {
     req_status_show on;
     }
     }
     }

## 3. 指令

- req_status_zone
- 语法: req_status_zone name string size
- 默认值: None
- 配置块: http
- 定义请求状态ZONE,请求按照string分组来排列，例如：
- req_status_zone server_url  $server_name$uri 256k;
- 域名+uri将会形成一条数据，可以看到所有url的带宽，流量，访问数
- req_status
- 语法: req_status zone1[ zone2]
- 默认值: None
- 配置块: http, server, location
- 在location中启用请求状态，你可以指定更多zones。
- req_status_show
- 语法: req_status_show on
- 默认值: None
- 配置块: location
- 展示数据

### 实时记录请求状态信息

`ngx_realtime_request`是nginx用来统计虚拟主机流量的模块, 首先和大家说下这个模块是基于域名的，将会记录这个域名的请求量、发送字节、返回http状态码的数量，特性如下：

- 基于域名记录
- 记录请求数据量
- 记录发送、响应流量
- 记录返回各种http状态码统计数据


## 1.  安装

    # cd /usr/local/src/
    # wget "http://nginx.org/download/nginx-1.4.2.tar.gz"
    # tar -xzvf nginx-1.4.2.tar.gz
    # wget https://github.com/magicbear/ngx_realtime_request_module/archive/master.zip -O ngx_realtime_request.zip
    # unzip ngx_realtime_request.zip
    # cd nginx-1.4.2/
    # ./configure --prefix=/usr/local/nginx-1.4.2 --add-module=../ngx_realtime_request_module-master
    # make
    # make install

### 2.  指令(directives)

- realtime_zonesize
- 语法: realtime_zonesize size
- 默认值: 4m
- 配置块: http
- 设置slab大小
- realtime_request
- 语法: realtime_request [on/off]
- 默认值: none
- 配置块: location
- 开启统计


### 3.  配置实例

    http {
      realtime_zonesize 16m;
      
      server {
    server_name www.jb51.net
    location ~ /ttlsa-rt-status {
      realtime_request on;
    }
      }
    }