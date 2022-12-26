# docker-nginx-certbot

:link: <https://github.com/JonasAlfredsson/docker-nginx-certbot>

使用[Let's Encrypt][1]免费证书颁发机构及其客户端[_certbot_][2]自动创建和更新网站 SSL 证书。
构建在[官方 Nginx Docker 镜像][9](Debian 和 Alpine)之上，并使用 OpenSSL/LibreSSL 自动创建一些密码初始握手时使用的 Diffie-Hellman 参数。

> :information_source: 这个容器第一次启动时，可能需要很长时间才能响应请求。
> 更多信息请参见[Diffie-Hellman parameters](./docs/good_to_know.md#diffie-hellman-parameters)部分。

> :information_source: 在进行 Docker pull 操作时，请使用[特定标签](./docs/dockerhub_tags.md)，因为`:latest` 可能并不总是 100%稳定。

### 值得注意的特性

- 当[请求证书](./docs/good_to_know.md#how-the-script-add-domain-names-to-certificate-requests) (i.e. both `example.com` and `www.example.com`)时处理多个服务器名.
- 处理通配符域请求，以防您使用[DNS 身份验证](./docs/certbot_authenticators.md).
- 可以[同时](./docs/advanced_usage.md#multi-certificate-setup)申请[RSA 和 ECDSA](./docs/good_to_know.md#ecdsa-and-rsa-certificates)证书
- 如果已定义，将创建[Diffie-Hellman 参数](./docs/good_to_know.md#diffie-hellman-parameters)。
- 使用[父容器][9]的[`/docker-entrypoint.d/`][7]文件夹。
- 当停止/杀死/失败时，将报告正确的[退出代码][6]。
- 你可以通过[发送一个`SIGHUP`](./docs/advanced_usage.md#manualforce-renewal)信号来重新加载配置(不需要重新启动容器)。
- 在[本地 CA](./docs/advanced_usage.md#local-ca)的帮助下**离线**使用此映像的可能性.
- 为[多个体系结构](./docs/dockerhub_tags.md)构建的[Debian 和 Alpine][14]映像.

## 使用

### 开始之前

1. 本指南希望您已经拥有一个指向正确 IP 地址的域，并且如果您使用 NAT，您的端口`80` and `443`都被正确转发。
   否则，我推荐[DuckDNS][12]作为动态 DNS 提供商，然后搜索如何在你的路由器上端口转发，或者可能找到它[这里][13]。

2. 我建议您至少阅读[Good to Know](./docs/good_to_know.md)文档中的前两部分，因为这将为您提供一些关于如何创建基本服务器配置以及如何使用 Let's Encrypt 预演服务器以不受速率限制的重要提示。

3. 我认为没有必要提及你是否找到了这个存储库，但你需要安装[Docker][11]才能正常工作。

## 可用的环境变量

### 必选

- `CERTBOT_EMAIL`: 你的电子邮件地址。Let's Encrypt 用于在出现安全问题时与您联系。

### 可选

- `DHPARAM_SIZE`: [Diffie-Hellman 参数](./docs/good_to_know.md#diffie-hellman-parameters)的大小 (default: `2048`)
- `ELLIPTIC_CURVE`: The size/[curve][15] of the ECDSA keys (default: `secp256r1`)
- `RENEWAL_INTERVAL`: Time interval between certbot's [renewal checks](./docs/good_to_know.md#renewal-check-interval) (default: `8d`)
- `RSA_KEY_SIZE`: The size of the RSA encryption keys (default: `2048`)
- `STAGING`: Set to `1` to use Let's Encrypt's [staging servers](./docs/good_to_know.md#initial-testing) (default: `0`)
- `USE_ECDSA`: Set to `0` to have certbot use [RSA instead of ECDSA](./docs/good_to_know.md#ecdsa-and-rsa-certificates) (default: `1`)

### 先进的

- `CERTBOT_AUTHENTICATOR`: The [authenticator plugin](./docs/certbot_authenticators.md) to use when responding to challenges (default: `webroot`)
- `CERTBOT_DNS_PROPAGATION_SECONDS`: The number of seconds to wait for the DNS challenge to [propagate](./docs/certbot_authenticators.md#troubleshooting-tips) (default: certbot's default)
- `DEBUG`: Set to `1` to enable debug messages and use the [`nginx-debug`][10] binary (default: `0`)
- `USE_LOCAL_CA`: Set to `1` to enable the use of a [local certificate authority](./docs/advanced_usage.md#local-ca) (default: `0`)

## 卷

- `/etc/letsencrypt`: 保存获取的证书和 Diffie-Hellman 参数

## 运行 `docker run`

Create your own [`user_conf.d/`](./docs/good_to_know.md#the-user_confd-folder)
folder and place all of you custom server config files in there. When done you
can just start the container with the following command
([available tags](./docs/dockerhub_tags.md)):

```bash
docker run -it -p 80:80 -p 443:443 \
           --env CERTBOT_EMAIL=your@email.org \
           -v $(pwd)/nginx_secrets:/etc/letsencrypt \
           -v $(pwd)/user_conf.d:/etc/nginx/user_conf.d:ro \
           --name nginx-certbot jonasal/nginx-certbot:latest
```

> You should be able to detach from the container by holding `Ctrl` and pressing
> `p` + `q` after each other.

As was mentioned in the introduction; the very first time this container is
started it might take a long time before before it is ready to
[respond to requests](./docs/good_to_know.md#diffie-hellman-parameters), please
be a little bit patient. If you change any of the config files after the
container is ready, you can just
[send in a `SIGHUP`](./docs/advanced_usage.md#manualforce-renewal) to tell
the scripts and Nginx to reload everything.

```bash
docker kill --signal=HUP <container_name>
```

## 运行 `docker-compose`

An example of a [`docker-compose.yaml`](./examples/docker-compose.yml) file can
be found in the [`examples/`](./examples) folder. The default parameters that
are found inside the [`nginx-certbot.env`](./examples/nginx-certbot.env) file
will be overwritten by any environment variables you set inside the `.yaml`
file.

> NOTE: You can use both `environment:` and `env_file:` together or only one

        of them, the only requirement is that `CERTBOT_EMAIL` is defined
        somewhere.

Like in the example above, you just need to place your custom server configs
inside your [`user_conf.d/`](./docs/good_to_know.md#the-user_confd-folder)
folder beforehand. Then you start it all with the following command.

```bash
docker-compose up
```

## 自己构建

如果你制作了自己的`Dockerfile`，这个选项是适用的。
在[本文档](./docs/dockerhub_tags.md)或[Docker Hub][8]上查看哪些标记可用，然后选择您想要的具体程度。

在这种情况下，完全可以跳过[`user_conf.d/`]文件夹，直接将文件写入 Nginx 的`conf.d/`文件夹。
通过这种方式，您可以用自己的文件替换我构建的[到映像](./src/nginx_conf.d/)。
但是，如果您这样做，请花点时间来理解它们是做什么的，以及为了让 certbot 继续工作，您需要包括什么。

```Dockerfile
FROM jonasal/nginx-certbot:latest
COPY conf.d/* /etc/nginx/conf.d/
```

# 测试

我们使用[BATS][16]来测试这个代码库的部分内容。
运行所有测试的最简单方法是在此存储库的根目录中执行以下命令:

```bash
docker run -it --rm -v "$(pwd):/workdir" ffurrer/bats:latest ./tests
```

## 更多的资源

这里是其他资源的链接集合，可以提供有用的信息。

- [Good to Know](./docs/good_to_know.md): 了解这个图像和它提供的功能有很多好处。
- [Changelog](./docs/changelog.md): 此存储库的所有标记版本的列表，以及各个版本之间发生变化的项目符号。
- [DockerHub Tags](./docs/dockerhub_tags.md): Docker Hub 提供的所有标记。
- [Advanced Usage](./docs/advanced_usage.md): 有关此映像提供的更高级功能的信息。
- [Certbot Authenticators](./docs/certbot_authenticators.md): 关于此映像中可用的不同身份验证器的信息。
- [Nginx Tips](./docs/nginx_tips.md): 一些关于如何配置 Nginx 的有趣提示。

## 外部指南

Here is a list of projects that use this image in various creative ways. Take
a look and see if one of these helps or inspires you to do something similar:

- [A `Node.js` application served over HTTPS in AWS Elastic Beanstalk](https://efraim-rodrigues.medium.com/using-docker-to-containerize-your-node-js-aefcd1ecd37d)
- [Host your own `Nakama` server](https://www.snopekgames.com/tutorial/2021/how-host-nakama-server-10mo)

## 致谢及感谢

This container requests SSL certificates from [Let's Encrypt][1], with the help
of their [_certbot_][2] script, which they provide for the absolutely bargain
price of free! If you like what they do, please [donate][3].

This repository was originally forked from [`@henridwyer`][4] by
[`@staticfloat`][5], before it was forked again by me. However, the changes to
the code has since become so significant that this has now been detached as its
own independent repository (while still retaining all the history). Migration
instructions, from `@staticfloat`'s image, can be found
[here](./docs/good_to_know.md#help-migrating-from-staticfloats-image).

[1]: https://letsencrypt.org/
[2]: https://github.com/certbot/certbot
[3]: https://letsencrypt.org/donate/
[4]: https://github.com/henridwyer/docker-letsencrypt-cron
[5]: https://github.com/staticfloat/docker-nginx-certbot
[6]: https://github.com/JonasAlfredsson/docker-nginx-certbot/commit/43dde6ec24f399fe49729b28ba4892665e3d7078
[7]: https://github.com/nginxinc/docker-nginx/tree/master/entrypoint
[8]: https://hub.docker.com/r/jonasal/nginx-certbot
[9]: https://github.com/nginxinc/docker-nginx
[10]: https://github.com/docker-library/docs/tree/master/nginx#running-nginx-in-debug-mode
[11]: https://docs.docker.com/engine/install/
[12]: https://www.duckdns.org/
[13]: https://portforward.com/router.htm
[14]: https://github.com/JonasAlfredsson/docker-nginx-certbot/issues/28
[15]: https://security.stackexchange.com/a/104991
[16]: https://github.com/bats-core/bats-core
