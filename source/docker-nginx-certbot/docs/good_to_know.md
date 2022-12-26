# 知道就好

本文档包含在开始使用此图像之前最好了解的功能和行为信息。
请随意阅读，但我向大家推荐前两部分。

## 初始测试

如果你只是在尝试设置这个，我建议你设置环境变量`STAGING=1`，因为这会将 Let’s Encrypt 挑战 URL 更改为他们的 STAGING URL。
这不会为您提供*合适的*证书，但与非分期[生产证书][2]相比，它具有高得可笑的[速率限制][1]，因此您可以在不必担心的情况下犯更多错误。
您还可以添加 `DEBUG=1` 以获得更详细的日志记录，以便更好地理解正在发生的事情。

像这样引入他们:

```bash
docker run -it -p 80:80 -p 443:443 \
           --env CERTBOT_EMAIL=your@email.org \
           --env STAGING=1 \
           --env DEBUG=1 \
           jonasal/nginx-certbot:latest
```

!!! note

    请注意，当切换到生产证书时，您要么需要删除预演证书，要么发出[强制更新](./advanced_usage.md#manualforce-renewal)，
    因为默认情况下，如果任何有效的(预演或生产)证书已经存在，certbot 将不会请求新证书。

## 创建一个服务器的`.conf`文件

作为 Nginx 中一个基本的(但有效的)SSL 服务器的例子，你可以查看[`examples/`](../examples)目录中的 [`example_server.conf`](../examples/example_server.conf)文件。

```nginx title="../examples/example_server.conf"
--8<-- "example_server.conf"
```

通过将`yourdomain.org`替换为您自己的域名，您实际上可以使用此配置快速测试事情是否正常工作。
在实际执行此操作时，还应该将证书路径["test-name"](#how-the-script-add-domain-names-to-certificate-requests)更改为更具描述性的内容。

将修改后的配置放在[`user_conf.d/`](#the-user_confd-folder)文件夹中，然后按照[主 README](../README.md#run-with-docker-run)中描述的那样运行它。
让容器暂时发挥它的[魔力](#diffie-hellman-parameters)，然后尝试访问您的域。
现在，您应该看到字符串“`Let's Encrypt certificate successfully installed!`”。

容器的配置文件夹中[已经存在](../src/nginx_conf.d/)的文件用于处理所有传入请求(不属于 certbot 挑战请求)的 HTTPS 重定向，因此要注意不要覆盖这些文件，除非您知道自己在做什么。

## `user_conf.d` 文件夹

默认情况下，Nginx 将从`/etc/nginx/conf.d/`文件夹中加载任何以`.conf`结尾的文件。
然而，这个映像使用了两个重要的[配置文件](../src/nginx_conf.d/)，它们需要存在(除非你知道如何用你自己的文件替换它们)，并且主机挂载一个本地文件夹到前面提到的位置会掩盖这些重要文件。

为了解决这个问题，我建议你主机挂载一个本地文件夹到`/etc/nginx/user_conf.d/`，一部分管理脚本将[创建符号链接][3]从`conf.d/`到`user_conf.d/`中的文件。
通过这种方式，我们为用户提供了一种简单的方法来启动容器，而不必首先构建本地映像，同时仍然让他们有机会继续以旧的方式进行操作，例如[`@staticfloat` 镜像][5]的工作方式。

<a id="how-the-script-add-domain-names-to-certificate-requests"></a>

## 脚本如何向证书请求添加域名

所包含的脚本将遍历 Nginx `/etc/nginx/conf.d/`文件夹中找到的所有配置文件(_`.conf`_)，并从文件的内容创建请求。
在每个独特的文件中，它会找到任何一行说:

```
ssl_certificate_key /etc/letsencrypt/live/test-name/privkey.pem;
```

并且只提取这里说“`test-name`”的部分。
这是将提供给 certbot 的[`--cert-name`][14]参数的值，因此尽管您基本上可以在这里设置任何您想要的名称，但我建议您为了自己的缘故保持它的描述性。

在此之后，脚本将找到包含`server_name`的所有行，并在同一行中列出所有域名。
一个包含如下内容的文件:

```
server {
    listen              443 ssl;
    server_name         yourdomain.org www.yourdomain.org;
    ssl_certificate_key /etc/letsencrypt/live/test-name/privkey.pem;
    ...
}

server {
    listen              443 ssl;
    server_name         sub.yourdomain.org;
    ssl_certificate_key /etc/letsencrypt/live/test-name/privkey.pem;
    ...
}
```

将共享相同的证书文件(即"test-name" 证书)，并且所有列出的域变体将包括为有效的[alt names][15]。
也可以将这些服务器块分割为两个单独的配置文件，因为脚本将在整个扫描过程中跟踪"test-name"值，并将任何额外的发现添加到其中。
所以最后我们会得到一个请求，看起来像这样:

```
certbot --cert-name "test-name" ... -d yourdomain.org -d www.yourdomain.org -d sub.yourdomain.org
```

脚本在定义请求中应该包含的内容的可定制性方面非常强大，但这被认为是一个更高级的用例，可以在高级用法文档的[覆盖 `server_name`](./advanced_usage.md#override-server_name)部分中进一步研究。

此外，我们支持通配符域名，但这要求您使用能够进行 DNS-01 挑战的验证器，有关这方面的更多信息可以在[Certbot 认证](./certbot_authenticators.md)文档中找到。

<a id="ecdsa-and-rsa-certificates"></a>

## ECDSA 和 RSA 证书

[ECDSA(或 ECC)][16]证书使用比成熟的 RSA 证书更新的加密算法，并且据称更安全，同时更小。
它们的缺点是还没有被所有客户端支持，但是如果您不希望提供[Mozillas 兼容性表][17]中“Modern”行以外的任何服务，那么您应该毫不犹豫地配置 certbot 来请求这些类型的证书。

这可以通过设置[环境变量](../README.md#optional) `USE_ECDSA=1`(3.0.1 版本以来的默认值)来实现，并且您可以选择使用`ELLIPTIC_CURVE`来优化要使用的曲线。
如果你已经下载了 RSA 证书，你将不得不等待直到他们到期，或[强制](./advanced_usage.md#manualforce-renewal)更新，在此更改生效之前。

使用这个选项，你将只为你所有的服务器配置创建 ECDSA 证书，然而，我应该提到的是，有一种方法可以配置 Nginx 同时提供 ECDSA 和 RSA 证书，但这在[高级用法](./advanced_usage.md#multi-certificate-setup)文档中有进一步的解释。

## 续订检查周期

该容器将在环境变量`RENEWAL_INTERVAL`中定义的时间持续时间过后自动启动 certbot 证书更新检查。
在 certbot 完成它的工作后，代码将返回并等待定义的时间，然后再次触发。

这个过程非常简单，只是一个`while [ true ];`循环，结尾是`sleep`:

```bash
while [ true ]; do
    # Run certbot...
    sleep "$RENEWAL_INTERVAL"
done
```

因此，在设置环境变量时，可以使用`sleep`识别的任何字符串，例如`3600` or `60m` or `1h`。
在它的[手册][4]中阅读更多关于允许的值。

默认值是`8d`，因为这允许每月多次重试，同时将日志中的输出保持在非常低的水平。
如果没有什么需要更新，certbot 不会做任何事情，所以如果你想把它设置得更低应该是没有问题的。
唯一需要考虑的是不要让它超过一个月，因为这样您就会错过 certbot 认为有必要更新证书的窗口[6]。

## Diffie-Hellman 参数

关于 Diffie-Hellman 参数，建议你为你的服务器设置一个，在 Nginx 中，你通过在服务器块中包含以`ssl_dhparam`开头的行来定义它(参见 [`example_server.conf`](../examples/example_server.conf))。
然而，你可以在没有它的情况下创建一个配置文件，Nginx 可以很好地处理不依赖于 Diffie-Hellman 密钥交换的密码([关于密码的更多信息][7])。

```nginx title="../examples/example_server.conf"
--8<-- "example_server.conf"
```

这些参数越大，生成它们所需的时间就越长。
我很不幸，我花了 65 分钟在一个旧的 3.0GHz CPU 上生成一个 4096 位的参数。
这将在运行之间发生很大的变化，因为其中涉及到一些随机性。
一个 2048 位的参数，今天仍然是安全的，在现代 CPU 上大约可以在 1-3 分钟内计算出来(这个过程只需要完成一次，因为这些参数中的一个在你网站的剩余生命周期中都是有效的)。
要修改参数的大小，可以设置`DHPARAM_SIZE`环境变量。
如果没有提供任何信息，默认为`2048`。

It is also possible to have **all** your server configs point to **the same**
Diffie-Hellman parameter on disk. There is no negative effects in doing this for
home use ([source 1][8] & [source 2][9]). For persistence you should place it
inside the dedicated folder `/etc/letsencrypt/dhparams/`, which is inside the
predefined Docker [volume](../README.md#volumes). There is, however, no
requirement to do so, since a missing parameter will be created where the
config file expects the file to be. But this would mean that the script will
have to re-create these every time you restart the container, which may become
a little bit tedious.

You can also create this file on a completely different (faster?) computer and
just mount/copy the created file into this container. This is perfectly fine,
since it is nothing "private/personal" about this file. The only thing to
think about in that case would perhaps be to use a folder that is not under
`/etc/letsencrypt/`, since that would otherwise cause a double mount.

## 帮助从`@staticfloat`的映像迁移

当涉及到构建/运行时，这两个映像并没有什么不同，因为这个存储库最初是一个分支。
所以就像在`@staticfloat`的设置中一样，你需要把你自己的`*.conf`文件放到容器的`/etc/nginx/conf.d/`文件夹中，然后你应该可以像启动他的一样启动这个文件。

This can either be done by copying your own files into the container at [build time](../README.md#build-it-yourself), or you can mount a local folder to [`/etc/nginx/user_conf.d/`](#the-user_confd-folder) and
[run it directly](../README.md#run-with-docker-run).
In the former case you need to make sure you do not accidentally overwrite the two files present in this
repository's [`nginx_conf.d/`](../src/nginx_conf.d/) folder, since these are required in order for certbot to request certificates.

The only obligatory environment variable for starting this container is the [`CERTBOT_EMAIL`](../README.md#required) one, just like in `@staticfloat`'s
case, but I have exposed a [couple of more](../README.md#optional) that can be changed from their defaults if you like. Then there is of course any environment variables read by the [parent container][11] as well, but those are probably not as important.

If you were using [templating][12] before, you should probably look into ["template" files][13] used by the Nginx parent container, since this is not something I have personally implemented in mine.

[1]: https://letsencrypt.org/docs/staging-environment/
[2]: https://letsencrypt.org/docs/rate-limits/
[3]: https://github.com/JonasAlfredsson/docker-nginx-certbot/commit/91f8ecaa613f1e7c0dc4ece38fa8f38a004f61ec
[4]: http://man7.org/linux/man-pages/man1/sleep.1.html
[5]: https://github.com/staticfloat/docker-nginx-certbot
[6]: https://community.letsencrypt.org/t/solved-how-often-to-renew/13678
[7]: https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html
[8]: https://security.stackexchange.com/questions/70831/does-dh-parameter-file-need-to-be-unique-per-private-key
[9]: https://security.stackexchange.com/questions/94390/whats-the-purpose-of-dh-parameters
[11]: https://github.com/nginxinc/docker-nginx
[12]: https://github.com/staticfloat/docker-nginx-certbot#templating
[13]: https://github.com/docker-library/docs/tree/master/nginx#using-environment-variables-in-nginx-configuration-new-in-119
[14]: https://certbot.eff.org/docs/using.html#where-are-my-certificates
[15]: https://www.digicert.com/faq/subject-alternative-name.htm
[16]: https://sectigostore.com/blog/ecdsa-vs-rsa-everything-you-need-to-know/
[17]: https://wiki.mozilla.org/Security/Server_Side_TLS
[18]: https://security.stackexchange.com/questions/31772/what-elliptic-curves-are-supported-by-browsers/104991#104991
