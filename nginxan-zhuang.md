#### 在linux服务器安装

```
rpm -ivh nginx-1.12.1-1.el7.ngx.x86_64.rpm
```

#### 查看配置文件位置

```
nginx -V
```

#### 修改nginx.conf

这个配置文件的位置，一般情况下位于/etc/nginx/nginx.conf，可使用nginx -V查看

使用vim打开后，将user和worker\_processes进行如下修改

```
user root
worker_processes auto
```

#### nginx启动、停止、重启

* 启动

```
service start nginx
```

* 停止

```
service stop nginx
```

* 重启

```
service restart nginx
```

#### 配置文件结构

> 在修改nginx.conf文件后，我们需要进入conf文件夹下（一般位于/etc/nginx/conf.d/），寻找或新建\*.conf文件

nginx 是由一些模块组成，我们一般在配置文件中使用一些具体的指令来控制它们。指令被分为简单指令和块级命令。一个简单的指令是由名字和参数组成，中间用空格分开，并以分号结尾。比如：

```
root /data/www
```

而块级指令和简单指令有着一样的结构，但是末尾不是分号而是用`{` 和 `}`大括号包裹的额外指令集如果一个块级指令的大括号里有其他指令，则它被叫做一个上下文（比如：[events](http://nginx.org/en/docs/ngx_core_module.html#events)，[http](http://nginx.org/en/docs/http/ngx_http_core_module.html#httph)，[server](http://nginx.org/en/docs/http/ngx_http_core_module.html#servers)，和[location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)）。`events`和`http`的指令是放在主上下文中，`server`放在`http`中,`location`放在`server`中。

```
http {
    server { 
         listen       80;
         server_name  domain1.com www.domain1.com;
         access_log   logs/domain1.access.log  main;
         root         html;

         location ~ \.php$ {
           fastcgi_pass   127.0.0.1:1025;
         }
    }
}
```



