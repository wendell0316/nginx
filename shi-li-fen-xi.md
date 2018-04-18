# newlook平台nginx配置简述

> 查看本文时建议对比newlook的nginx配置。

* ### upstream

  定义一个服务器集群，可被[proxy\_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass),[fastcgi\_pass](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass),[uwsgi\_pass](http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass),[scgi\_pass](http://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass), 和 [memcached\_pass](http://nginx.org/en/docs/http/ngx_http_memcached_module.html#memcached_pass)指令引用

```
upstream itoa_server {
  server 192.168.32.24:28899;
  server 192.168.32.25:28899;
  server 192.168.32.26:28899;
}
```

> 请求将被上面三个服务依次响应

* #### server

```
server {
  listen 80 default_server;
  server_name sharplook.eoitek.net newlook.eoitek.net;
  charset utf-8;
  return 301 https://$host$request_uri;
}
```

> 表示监听 http://sharplook.eoitek.net或http://newlook.eoitek.net的默认80端口，将永久重定向到对应的https

```
server {
  listen 443 ssl http2 spdy fastopen=5 reuseport;
  server_name sharplook.eoitek.net newlook.eoitek.net;
  charset utf-8;

  root /opt/newlook;
  ssl_certificate /etc/nginx/ssl/sharplook.eoitek.net.cert.pem;
  ssl_certificate_key /etc/nginx/ssl/sharplook.eoitek.net.key.pem;
  ssl_trusted_certificate /etc/nginx/ssl/sharplook.eoitek.net.ca.pem;


}
```

> 监听[https://sharplook.eoitek.net或https://newlook.eoitek.net的默认443端口，且设置ssl证书。](https://sharplook.eoitek.net或https://newlook.eoitek.net的默认443端口，且设置ssl证书。)
>
> > root  表示所有请求的根目录

* #### location

`location`放在`server`中。`location`指令分为两种匹配模式：

1. 普通字符串匹配：以=开头或开头无引导字符（～）的规则
2. 正则匹配：以～或～\*开头表示正则匹配，~\*表示正则不区分大小写

在`server`模块中可以定义多个`location`指令来匹配不同的url请求，多个不同location配置的URI匹配模式，总体的匹配原则是：

**先匹配普通字符串模式，再匹配正则模式**。只识别URI部份，例如请求为：/test/abc/user.do?name=xxxx

一个请求过来后，Nginx匹配这个请求的流程如下：

* 先查找是否有=开头的精确匹配，如：location = /test/abc/user.do { … }

* 再查找普通匹配，以 最大前缀 为原则，如有以下两个location，则会匹配后一项

```
   location /test/ { … }

   location /test/abc { … }
```

* 匹配到一个普通格式后，搜索并未结束，而是暂存当前匹配的结果，并继续搜索正则匹配模式

* 所有正则匹配模式`location`中找到第一个匹配项后，就以此项为最终匹配结果\(所以正则匹配项匹配规则，受定义的前后顺序影响，但普通匹配模式不会\)

* 如果未找到正则匹配项，则以3中缓存的结果为最终匹配结果

* 如果一个匹配都没搜索到，则返回404

```
  location / {
    try_files $uri $uri/ /index.html =404;
    expires -1;
    etag off;
    if_modified_since off;
    more_clear_headers "Last-Modified";
    more_set_headers "Strict-Transport-Security max-age=86400";
    add_header Link "</api/itoa/config/advance>; rel=preload; as=fetch";
    add_header Link "</api/itoa/isldapauth>; rel=preload; as=fetch";
  }
```

> 该`location`会匹配所有以‘/’为前缀的请求，前提是在之前定义的有较大前缀的location中匹配不成功。
>
> > `try\_files`指令表示尝试寻找请求的目录，如果后面的三个参数都未完成匹配，则返回404页面
> >
> > `expires`指令表示匹配到的文件在浏览器的缓存时间，-1表示不缓存

```
  location ^~ /api/itoa {
    rewrite '^/api(.*)' $1;
    expires off;
    proxy_pass http://itoa_server;
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host  $host:$server_port;
    proxy_set_header X-Forwarded-Proto $scheme;
    break;
  }
```

> 在`rewrite`指令集中，若没有 `last`\(停止处理当前的ngx\_http\_rewrite\_module的指令集，并开始搜索与更改后的uri相匹配的location\)、`break`\(停止处理当前的ngx\_http\_rewrite\_module的指令集\)、`redirect`\(返回302临时重定向\)、或`permanent`\(返回301永久重定向\)
>
> > 该`location`块旨在将   /api/\***  改为   /**\*\*\* （真实后台接口无api路径）。然后以修改后的uri去请求接口。



