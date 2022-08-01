# Nginx安装(精简版)

安装gcc

```
yum install -y gcc
```

安装perl库

```
yum install -y pcre pcre-devel
```

安装zlib库

```
yum install -y zlib zlib-devel
```

 

编译安装 

```
./configure --prefix=/usr/local/nginx
或
./configure --prefix=/home/software/webserver 
或
./configure --prefix=/home/dcp/nginx/webserver --add-module=/home/software/lua-nginx-module --add-module=/home/dcp/ngx_devel_kit-0.3.0 --add-module=/home/dcp/ngx_openresty-1.7.7.2
 
make
make install
```

 

启动Nginx

进入安装好的目录 /usr/local/nginx/sbin

```
./nginx 启动
./nginx -s stop 快速停止
./nginx -s quit 优雅关闭，在退出前完成已经接受的连接请求 
./nginx -s reload 重新加载配置
```

# 配置Https

查看nginx内容

```
./sbin/nginx -V
--prefix=/home/software/nginx/webserver/
```

备份原sbin/nginx，并改名nginxold

进入nginx安装目录

```
./configure --prefix=/home/software/nginx/webserver/ --with-http_ssl_module
make
```

配置nginx.conf

```
server {
    	   listen          443 ssl;
    	   server_name     www.2021cn.top;
    	   
    	   ssl_certificate      /home/software/7851214_www.2021cn.top.pem;
    	   ssl_certificate_key    /home/software/7851214_www.2021cn.top.key;

    	   ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

    	   location / {
    	       root /home/studyboke;
    	       index index.html index.htm;
            #root   html;
            #index  bingdundun.html bingdundun.htm;
        }

        location /love {
            root   /home/software/page/;
        }

        location /pdf {
            proxy_redirect          off;
            proxy_set_header        Host $host;
            proxy_set_header        X-Real-IP $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    50m;
            proxy_pass   http://localhost:9002/;
            #rewrite  ^/(.*)$  http://localhost:9002/upload permanent;
        }

        location /kibana {
            proxy_pass   https://localhost:5601/app/home/;
        }

}
```

安装完重启nginx

安装防火墙

```java
yum install firewalld systemd -y

systemctl start firewalld

systemctl status firewalld
```

