Nginx

是一款网页服务器

OpenResty加上了插件的Nginx

nginx.exe 默认80端口

nginx.exe -s reload

```nginx
server {
    listen	80;
    server_name	localhost;
    default_type	text/html;
    location / {
        echo "hello nginx";
    }
    location = /a {
        echo "= /a";
    }
    location ^~ /a {
        echo "^~ /a";
    }
    location ~ /\w {
        echo "~ /\w";
    }
}
```

echo是OpenResty自带的插件

匹配是有优先级的 

/a是最高的全等于

^~a是第二高的匹配开头

~是第三优先级表示正则表达式 /\w表示字母数字

无特殊符号是第四优先级

相同优先级匹配度高的会展示

写在上面的会优先级高

```nginx
location /a {
    proxy_pass http://192.168.0.12:80;
}
匹配/a时会直接打到 http://192.168.0.12:80
```

反向代理：

```nginx
location /a {
    proxy_pass http://ip;
}
location /b/ {
    proxy_pass http://ip/;
}
上述配置会导致
/a/x ->http://ip/a/x;
/b/x -> http://ip/x;
```

负载均衡

```nginx
upstream group1 {
    server 192.168.0.12:80 weight=10
    server 192.168.0.12:81 weight=1
}
location /a {
    proxy_pass http://group1/
}
```

group1会根据权重分配访问