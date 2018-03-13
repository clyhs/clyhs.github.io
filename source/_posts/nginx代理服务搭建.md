---
title: nginx代理服务搭建
date: 2018-03-13 09:37:50
category: nginx
tags: [nginx,tomcat]
---

### 准备两个TOMCAT＋NGINX
* tomcat1 192.168.0.100:8080
* tomcat2 192.168.0.100:8070
* nginx   192.168.0.100:80

### 启动两台TOMCAT
* 略

### 配置nginx.conf
```shell
# cd /etc/nginx
# vi nginx.conf


#user  nobody;
worker_processes  4;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  2048;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    #client_header_buffer_size 1024K;
    #large_client_header_buffers 4 1024K;
      


   # proxy_connect_timeout   300; 
   # proxy_send_timeout      300; 
   # proxy_read_timeout      300; 
   # proxy_buffer_size       16k; 
   # proxy_buffers           4 64k; 
   # proxy_busy_buffers_size 128k; 
   # proxy_temp_file_write_size 128k;


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
        server 192.168.0.100:8080 max_fails=3 weight=1 fail_timeout=300s;
        #server 192.168.0.100:8080 max_fails=3 weight=1 fail_timeout=300s;
        keepalive 256;
    }

    server {
        listen       80;
        server_name  localhost;
        
        #client_max_body_size 64m;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;
         
        client_header_buffer_size 2m;
        large_client_header_buffers 4 1m;        
 
        charset utf-8;
        #
        location / {
            proxy_pass http://backend_tomcats;
            proxy_set_header HOST $host:80;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
	

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
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

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}


```
### 访问地址：
* http://192.168.0.100
