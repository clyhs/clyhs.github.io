---
title: nginx的几种负载均衡
date: 2019-05-30 11:19:45
category: nginx
tags: nginx
---

### nginx的几种负载均衡

* （1）轮询(默认):每个请求按时间顺序逐一分配到不同的后端服务器;

  ```shell
  upstream backend_tomcats {
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
  }
  ```

  

* （2）ip_hash:每个请求按访问IP的hash结果分配，同一个IP客户端固定访问一个后端服务器。可以保证来自同一ip的请求被打到固定的机器上，可以解决session问题。

  ```
  upstream backend_tomcats {
          ip_hash;
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
  }
  ```

  *这种方法，做负载均衡就没有意义，因为他会直接压到一台机，另一台没起到集群负载的作用，最好把session存到redis上*

* （3）url_hash:按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器。后台服务器为缓存的时候效率。

  ```shell
  upstream backend_tomcats {
          server xx.x.x.x:8080 ;
          server xx.x.x.x:8080 ;
          hash $request_uri; 
          hash_method crc32; 
  }
  ```

  *注意：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法。*

* (4) fair:这是比上面两个更加智能的负载均衡算法。
  此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持 fair的，如果需要使用这种调度算法，必须下载Nginx的 upstream_fair模块

  ```shell
  upstream backend_tomcats {
          fair;
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
  }
  ```

* (5)least_conn：选取活跃连接数与权重weight的比值最小者为下一个处理请求的server

  ```shell
  upstream backend_tomcats {
          least_conn;
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
          server xx.x.x.x:8080 max_fails=3 weight=1 fail_timeout=30s;
  }
  ```

*下面几个名词解释说明*

**down** 表示单前的server暂时不参与负载.

**weight** 默认为1.weight越大，负载的权重就越大。

**max_fails** ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.

**fail_timeout** : max_fails次失败后，暂停的时间。

**backup**： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。



### nginx1.12.1.tar.gz安装

先安装如下包：

cpp gcc gcc-c++ glibc-devel glibc-headers libstdc++ kernel-headers keyutils-lib-devel krb5-devel libmpc libselinux-devel libsepol-devel libverto-devel libcom_err-devel
zlib zlib-devel openssl openssl-devel pcre pcre-devel

下载upstream-fair包地址：https://github.com/gnosek/nginx-upstream-fair

更名为upstream放到/home目录下

安装命令如下：

```shell

./configure --prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-log-path=/var/log/nginx/access.log --http-proxy-temp-path=/var/lib/nginx/proxy --lock-path=/var/lock/nginx.lock --pid-path=/var/run/nginx.pid --with-debug --with-http_dav_module --with-http_flv_module --with-http_geoip_module --with-http_gzip_static_module --with-http_realip_module --with-http_stub_status_module --with-http_ssl_module --with-http_sub_module --with-ipv6 --with-mail --with-mail_ssl_module --add-module=/home/upstream/
make && make install
```



### nginx参数配置

附一个完整的nginx.conf的配置：

```shell

#user  root;
worker_processes  4;
worker_rlimit_nofile 65535;
events {
    use epoll;
    worker_connections  204800;
    multi_accept on;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    #页面缓存目录，注意目录权限要777
    proxy_temp_path /home/domains/pascloud/nginx/proxy_temp;
    proxy_cache_path /home/domains/pascloud/nginx/proxy_cache levels=1:2 keys_zone=cache_one:10240m inactive=10d max_size=2000m;


    sendfile        on;
    tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    #
    gzip on;
    gzip_static on;  
    gzip_comp_level 9;
    gzip_min_length 1400;
    gzip_vary  on;
    gzip_http_version 1.1;  
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css text/xml text/javascript image/gif image/jpeg application/x-javascript application/xml;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";
    #
    client_max_body_size 8m;
    client_body_buffer_size 512k;
    #
    upstream backend_tomcats {
        #least_conn;
        #ip_hash;
        fair;
	    server xxxx:8080 max_fails=1 weight=2 fail_timeout=20s;
        server xxxx:8080 max_fails=1 weight=2 fail_timeout=20s;
    }

    server {
        listen       80;
        server_name  localhost;
        
         
        client_header_buffer_size 2m;
        large_client_header_buffers 4 1m;        
 
        charset utf-8;
        #

        access_log off;
        error_log off;
        location / {
            
            proxy_pass http://backend_tomcats;
            proxy_set_header HOST $host:80;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #proxy_http_version 1.1;
            #proxy_set_header Connection "";
            proxy_next_upstream error timeout invalid_header http_500 http_503 http_404;

            proxy_connect_timeout   300; 
            proxy_send_timeout      300; 
            proxy_read_timeout      600; 
            proxy_buffer_size       256k; 
            proxy_buffers           4 256k; 
            proxy_busy_buffers_size 256k; 
            proxy_temp_file_write_size 256k;
            proxy_max_temp_file_size 128m;

            
            proxy_cache cache_one ;
            proxy_cache_valid 200 304 12h ;
            proxy_cache_valid 301 302 1m ;
            proxy_cache_valid any 1m ;
            proxy_cache_key $host$uri$is_args$args;
            #proxy_ignore_headers Set-Cookie Cache-Control;
	        #proxy_hide_header Cache-Control;
        }
        #静态文件分离
        location ~* \.(gif|jpg|jpeg|png|js|css)$ {         
            root /home/domains/pascloud/ROOT/;
	        access_log off;
	        expires 10d;
            break;
	    }
	
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        

    }
}


```











