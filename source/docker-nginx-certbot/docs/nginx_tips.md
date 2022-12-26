# Nginx Tips

这个文档包含了一些关于如何以不同的方式修改 Nginx 的提示，可能会让你感兴趣。
这些都不是必须要做的，但知道这些信息更有用，我发现这些信息对任何潜在的未来努力都是有用的。

## Nginx 如何加载配置

为了理解 Nginx 如何加载任何自定义配置，我们首先要看一下父镜像中的主文件`nginx.conf`。
它包含了一些标准设置，但在最后一行，我们可以看到它打开`/etc/nginx/conf.d/`文件夹，并加载任何以`.conf`结尾的文件。

```bash
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;    # <------------ Extra stuff loaded here
}
```

Files in this folder are being loaded in alphabetical order, so something named
`00-proxy.conf` will be loaded before `10-other.conf`. This i really useful to
know, since it allows you to load common settings used by multiple `server`
blocks that are loaded afterwards.

However, all of these `.conf` file are loaded within the `http` block in Nginx,
so if you want to change anything outside of this block (e.g. `events`) you
will have to add some sort of [`/docker-entrypoint.d/`][7] script to handle it
before Nginx starts, or you can mount your own custom `nginx.conf` on top of
the default.

A small disclaimer on the last part is that a host mounted file
(`-v $(pwd)/nginx.conf:/etc/nginx/nginx.conf`) will [not change][8] inside the
container if it is changed on the host. However, if you host mount a directory,
and change any of the files within it, the changes will be visible inside the
container.

## 配置继承

为了使这个解释简单而有用，我们首先声明 Nginx 配置被分成四个块。
在外部块(例如 Global 块)中声明的变量和设置将被内部块(例如 Server 块)继承，除非你在这个内部块中更改它。

So in the example below I have added comments with the current value of the
[`keepalive_timeout`][9] setting in each block:

```bash
# -- Global/main block --
# keepalive_timeout = 60 (The default value)
http {
    # -- HTTP block --
    keepalive_timeout = 30 # The value has now changed to 30
    server {
        # -- Server block --
        # keepalive_timeout = 30 (value inherited from http block)
        location /abc/ {
            # -- Location block nbr 1 --
            keepalive_timeout = 50 # The value has now changed to 50
        }
        location /xyz/ {
            # -- Location block nbr 2 --
            # keepalive_timeout = 30 (value inherited from server block)
        }
    }
}
```

This is pretty straight forward for the settings that are only one value, but
the commonly used [`proxy_set_header`][10] setting can be declared multiple
times in order to add multiple values to it, and [its inheritance][11] works a
bit differently. The following is true of all of the settings that can be
declared multiple times.

In the example below we want to add two headers to all requests, so we
declare them in the `http` block. This builds a map/dictionary with the
key-value pairs we want, and this will be inherited to all the location blocks.
However, in the first location block we want to **add** another header, but
doing it in this way will instead overwrite the current one with just this new
header.

```bash
http {
    proxy_set_header key1 value1;
    proxy_set_header key2 value2;
    server {
        # proxy_headers: {
        #     "key1": "value1"
        #     "key2": "value2"
        # }
        location /abc/ {
            proxy_set_header key3 value3;
            # proxy_headers: {
            #     "key3": "value3"
            # }
        }
        location /xyz/ {
            # proxy_headers: {
            #     "key1": "value1"
            #     "key2": "value2"
            # }
        }
    }
}
```

The suggested solution to this problem is to create a separate file with the
"common" headers, and then `include` this file where needed. So in our case we
create the file `/etc/nginx/common_headers` with the following content:

```
proxy_set_header key1 value1;
proxy_set_header key2 value2;
```

and then change the config to the following which would make the special
location block have all the desired headers:

```bash
http {
    include common_headers;
    server {
        location /abc/ {
            include common_headers;
            proxy_set_header key3 value3;
            # proxy_headers: {
            #     "key1": "value1"
            #     "key2": "value2"
            #     "key3": "value3"
            # }
        }
        location /xyz/ {
        }
    }
}
```

## 拒绝未知服务器名

When setting up server blocks there exist a setting called `default_server`,
which means that Nginx will use this server block in case it cannot match
the incoming domain name with any of the other `server_name`s in its available
config files. However, a less known fact is that if you do not specify a
`default_server` Nginx will automatically use the [first server block][1] in
its configuration files as the default server.

This might cause confusion as Nginx could now "accidentally" serve a
completely wrong site without the user knowing it. Luckily HTTPS removes some
of this worry, since the browser will most likely throw an
`SSL_ERROR_BAD_CERT_DOMAIN` if the returned certificate is not valid for the
domain that the browser expected to visit. But if the cert is valid for that
domain as well, then there will be problems.

If you want to guard yourself against this, and return an error in the case
that the client tries to connect with an unknown server name, you need to
configure a catch-all block that responds in the default case. This is simple
in the non-SSL case, where you can just return `444` which will terminate the
connection immediately.

```
server {
    listen      80 default_server;
    server_name _;
    return      444;
}
```

> NOTE: The [redirector.conf](../src/nginx_conf.d/redirector.conf) should be

        the `default_server` for port 80 in this image.

Unfortunately it is not as simple in the secure HTTPS case, since Nginx would
first need to perform the SSL handshake (which needs a valid certificate)
before it can respond with `444` and drop the connection. To work around this
I found a comment in [this][2] post which mentions that in version `>=1.19.4`
of Nginx you can actually use the [`ssl_reject_handshake`][3] feature to
achieve the same functionality.

```
server {
    listen               443 ssl default_server;
    ssl_reject_handshake on;
}
```

This will lead to an `SSL_ERROR_UNRECOGNIZED_NAME_ALERT` error in case the
client tries to connect over HTTPS to a server name that is not served by this
instance of Nginx, and the connection will be dropped immediately.

## 添加自定义模块

Adding a [custom module][4] to Nginx is not enirely trivial, since most guides
I have found require you to re-complie everything with the desired module
included and thus you cannot make use of the official Docker image to build
upon. However, after some research I found that most of these modules are
possible to compile and load as a [dynamic module][5], which enables us to more
or less just add one file and then change one line in the main `nginx.conf`.

A complete example of how to do this is available over at
[AxisCommunications/docker-nginx-ldap][6], where a multi-stage Docker build
can be viewed that add the LDAP module to the official Nginx image with
minimal changes to the original.

[1]: https://nginx.org/en/docs/http/request_processing.html
[2]: https://serverfault.com/a/631073
[3]: https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_reject_handshake
[4]: https://www.nginx.com/resources/wiki/modules/
[5]: https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/
[6]: https://github.com/AxisCommunications/docker-nginx-ldap
[7]: https://github.com/nginxinc/docker-nginx/tree/master/entrypoint
[8]: hhttps://medium.com/@jonsbun/why-need-to-be-careful-when-mounting-single-files-into-a-docker-container-4f929340834
[9]: https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive_timeout
[10]: https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header
[11]: https://stackoverflow.com/a/32126596
