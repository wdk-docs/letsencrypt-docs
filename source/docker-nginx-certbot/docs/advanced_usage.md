# 高级用法

这个文档包含了关于被认为是“高级”的特性的信息，并且很可能需要你阅读一些实际的代码来完全理解发生了什么。

<a id="manualforce-renewal"></a>

## 手动/强制 更新

手动触发证书的更新可能会很有趣，这就是为什么 [`run_certbot.sh`](../src/scripts/run_certbot.sh)脚本可以在任何时候从容器中独立运行的原因。

然而，请求重新加载所有配置文件的首选方法是向容器发送[`SIGHUP`][1] :

```bash
docker kill --signal=HUP <container_name>
```

这将终止[睡眠计时器](./good_to_know.md#renewal-check-interval)，并使更新循环从头开始，其中包括许多其他检查，而不仅仅是证书。

虽然在大多数情况下这就足够了，但有时可能需要**强制**更新证书，即使 certbot 认为它可以保留证书一段时间(就像[这个][2]发生时)。
因此，当调用[`run_certbot.sh`](../src/scripts/run_certbot.sh)脚本时，可以添加`force`作为参数，使其将`--force-renewal`标记附加到所发出的请求上。

```bash
docker exec -it <container_name> /scripts/run_certbot.sh force
```

这将请求新的证书，而不管它们何时被设置为过期。

!!! warning "使用`force`会对**所有**证书发出新的请求，所以不要经常运行它，因为请求[生产证书][3]是有限制的。"

<a id=“override-server_name”></a>

## 覆盖 `server_name`

Nginx 允许你在[`server_name` declaration][18]中做很多事情，但由于这个映像中的脚本从相同的行中组成证书请求，我们在这些行中可能定义的内容受到严重限制。
例如，行`server_name mail.*`将产生一个域名`mail.*`的证书请求，该域名无效。

然而，为了克服这种限制，可以在同一行上定义一个特殊的注释，以覆盖脚本将拾取的内容。
在这个人为的例子中

```nginx
server {
    listen              443 ssl;
    ssl_certificate_key /etc/letsencrypt/live/test-name/privkey.pem;

    server_name         yourdomain.org;
    server_name         www.yourdomain.org; # certbot_domain:*.yourdomain.org
    server_name         sub.yourdomain.org; # certbot_domain:*.yourdomain.org
    server_name         mail.*;             # certbot_domain:*.yourdomain.org
    server_name         ~^(?<user>.+)\.yourdomain\.org$;
    ...
}
```

我们将以一个证书请求结束，它看起来像这样:

```
certbot --cert-name "test-name" ... -d yourdomain.org -d *.yourdomain.org
```

第一个服务器名称将像往常一样被选中，而下面三个将被注释中的域名遮蔽，即`*.yourdomain.org`(重复的名称将在最终请求中删除)。

最后一个服务器名是特殊的，因为它是一个正则表达式，并且总是以`~`开头。
因为我们知道我们永远不能从一个以该字符开头的名称创建一个有效的请求，它们将总是被脚本忽略(后面的注释将优先而不是被忽略)。
更详细的示例可以在[`example_server_overrides.conf`](../examples/example_server_overrides.conf)中查看。.

```nginx title="../examples/example_server_overrides.conf"
--8<-- "example_server_overrides.conf"
```

重要的是要记住，这里我们定义了一个通配符域名(`*.yourdomain.org`中的 `*`)，这要求您使用能够进行 DNS-01 挑战的验证器，有关此的更多信息可以在[certbot_authenticators.md](./certbot_authenticators.md)文档中找到。

<a id="multi-certificate-setup"></a>

## 多证书设置

这是[Good to Know](./good_to_know.md) 文档中[RSA 和 ECDSA](./good_to_know.md#ecdsa-and-rsa-certificates)部分的延续，其中简要提到，实际上可以让 Nginx 同时服务于这两种证书类型，从而再次扩展对半旧设备的支持，同时还允许使用最新的加密。
[设置][4]稍微复杂一些，但是[`example_server_multicert.conf`](../examples/example_server_multicert.conf)文件应该被配置，因此您应该只需要编辑顶部的"yourdomain.org"语句。

```nginx title="../examples/example_server_multicert.conf"
--8<-- "example_server_multicert.conf"
```

它的工作原理是 Nginx 能够为每个服务器块[加载多个证书文件][5]，然后你按照优先选择 ECDSA 证书的顺序配置密码套件。
在容器内运行的[scripts](../src/scripts/run_certbot.sh)然后在[`--cert-name`](./good_to_know.md#how-the-script-add-domain-names-to-certificate-requests)参数中寻找这些字符串的一些(不区分大小写)变体:

- `-rsa`
- `.rsa`
- `-ecc`
- `.ecc`
- `-ecdsa`
- `.ecdsa`

并使用正确的类型集发出证书请求。
有关更多详细信息，请参阅[实际提交][6]，但您需要知道的是，如果发现这些选项，它们将覆盖[`USE_ECDSA`](../README.md#optional)环境变量。

## 使用自定义 ACME URL

在[`run_certbot.sh`](../src/scripts/run_certbot.sh)脚本的顶部有两个变量:

- `CERTBOT_PRODUCTION_URL`
- `CERTBOT_STAGING_URL`

它们用于定义在请求新证书时 certbot 将尝试联系哪个服务器。
这些变量具有默认值，但是可以通过定义具有相同名称的环境变量来覆盖它们。
这允许您将 certbot 重定向到另一个自定义 URL(例如，如果您正在运行自己的自定义 AMCE 服务器)。

## 本地 CA

在一个网站的开发阶段，你可能在一台没有指向自己的 DNS 记录的计算机上测试东西，或者它可能根本没有互联网访问。
由于 certbot 需要这两个条件才能正常工作，因此在这些特殊情况下，以前不可能使用这个图像。

这就是创建[`run_local_ca.sh`](../src/scripts/run_local_ca.sh) 脚本的原因，因为这使得使用[本地(自签名)证书颁发机构][10]可以在不依赖任何外部服务或互联网连接的情况下颁发网站证书成为可能。
它还使我们能够颁发对`localhost`和/或`::1`等 IP 地址有效的证书，否则 certbot [无法][7]创建这些地址。

To enable the usage of this local CA you just set
[`USE_LOCAL_CA=1`](../README.md#advanced), and this will then trigger the
execution of the [`run_local_ca.sh`](../src/scripts/run_local_ca.sh) script
instead of the [`run_certbot.sh`](../src/scripts/run_certbot.sh) one when it is
time to renew the certificates. This script, when run, will always overwrite
any previous keys and certificates, so alternating between the use of a local
CA and certbot without first emptying the `/etc/letsencrypt` folder is not
supported.

The script is designed to mimic certbot as closely as reasonable, so the
keys/certs created are placed in the same locations as certbot would have. This
means that you only have to edit the `server_name` in your server configuration
files to include the variant that you want for your local instance (e.g.
`localhost`) and you should be all set.

However, if you navigate to your site at this point you will run into an error
named similar to Firefox's `SEC_ERROR_UNKNOWN_ISSUER`, which just means that
your browser does not recognize the CA that has signed your site's certificate.
This is expected (since we just created our own local CA), but at this point
the connection is using all the fancy HTTPS stuff, and is thus "secure", so you
can just ignore this warning if you want.

Another solution is to [import][9] the local CA's certificate created by this
script into your browser, thus making this a known certificate authority and
any of its signed certs trusted. What this file is, and how to obtain it, is
explained further in the [next section](#files-and-folders).

### 文件和文件夹

本地 CA 要运行，需要做几件事:

- `caPrivkey.pem`: CA 使用的私钥->这是最机密的东西(在真实 CA 的情况下)，必须不惜一切代价保护它。
- `caCert.pem`: 所有客户端都需要 CA 的公共证书部分，以便信任此 CA 签署的任何其他证书。
- `serial.txt`: CA 每次签署新证书时加 1 的长随机十六进制数。
- `index.txt`: 保存此 CA 已颁发的所有证书的[记录][8]。
- `new_certs/`: 存放所有新签署证书副本的文件夹。

All of these are created automatically by the script inside the folder defined
by [`LOCAL_CA_DIR`](../src/scripts/run_local_ca.sh) (which defaults to
`/etc/local_ca`), so by host mounting this folder you will be able to see all
these files. By then taking the `caCert.pem` and [importing][9] it in your
browser you will be able to visit these sites without the error stating that
the certificate is signed by an unknown authority.

> The validity period for the automatically created CA is only 30 days, and the
> reason for this is to deter people from using this solution in production.

An important thing to know is that these files are only created if they do
not exist. What this enables is an even more advanced usecase where you might
already have a private key and certificate that you trust on your devices, so
you would like to continue using it for the websites you host as well. Read
more about this in the [next section](#creating-a-custom-ca).

### 创建自定义 CA

如前一节所述，可以为[`run_local_ca.sh`](../src/scripts/run_local_ca.sh)脚本提供您手动创建的本地证书颁发机构，该证书颁发机构的有效期可能超过 30 天，并且您希望在其他多个设备上信任该证书颁发机构。
这是一种解决方案，如果您想在永远无法通过开放互联网访问的服务上设置 HTTPS，但您仍然希望它们的通信是安全的，则可以使用该解决方案。

基本上，你所需要做的就是主机挂载你的自定义私钥和证书到`LOCAL_CA_DIR`，脚本将使用这些私钥和证书，而不是自动创建的短时间的私钥和证书。
只需确保文件的命名符合[脚本期望的](#files-and-folders)，如果这些组件中的任何一个缺失，它们将在服务第一次启动时创建。

> 到目前为止，不支持密码保护的私钥。

I did not find it trivial to create a **well configured** CA, so if you want
to go this route I really suggest that you read up on what you are doing and
making sure all settings are correctly tuned for your usecase.

There is a [lot][11] of [high-level][12] information [available][13] in regards
to how to create your own CA, but what I found most confusing was exactly what
was expected to be inside the [`openssl.cnf`][14] file that is necessary to
have when running most of the OpenSSL commands. The configuration that is
present inside the [`run_local_ca.sh`](../src/scripts/run_local_ca.sh) script
should be quite minimalistic for what we need, while still providing the
[strict settings][15] that some clients need else they will reject these
custom certificates.

The most comprehensive guide I have found is the [OpenSSL Cookbook][17],
which goes into great detail about basically everything OpenSSL is able to do,
along with [this post][16] which summarizes the settings needed for different
certificate types. With these two you should be able to make an informed
configuration in case you want to create your own custom certificate authority,
and you may of course take a look at the commands used in the
[`generate_ca()`](../src/scripts/run_local_ca.sh) function to help you on your
way of creating your own files.

[1]: https://github.com/JonasAlfredsson/docker-nginx-certbot/commit/bf2c1354f55adffadc13b1f1792e205f9dd25f86
[2]: https://community.letsencrypt.org/t/revoking-certain-certificates-on-march-4/114864
[3]: https://letsencrypt.org/docs/rate-limits/
[4]: https://medium.com/hackernoon/rsa-and-ecdsa-hybrid-nginx-setup-with-letsencrypt-certificates-ee422695d7d3
[5]: https://scotthelme.co.uk/hybrid-rsa-and-ecdsa-certificates-with-nginx/
[6]: https://github.com/JonasAlfredsson/docker-nginx-certbot/commit/9195bf02cb200dcec8206b46da971734b1d6669f
[7]: https://letsencrypt.org/docs/certificates-for-localhost/
[8]: https://pki-tutorial.readthedocs.io/en/latest/cadb.html
[9]: https://support.securly.com/hc/en-us/articles/360008547993-How-to-Install-Securly-s-SSL-Certificate-in-Firefox-on-Windows
[10]: https://gist.github.com/Soarez/9688998
[11]: https://gist.github.com/fntlnz/cf14feb5a46b2eda428e000157447309
[12]: https://github.com/llekn/openssl-ca
[13]: https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html
[14]: https://github.com/llekn/openssl-ca/blob/master/openssl.cnf
[15]: https://derflounder.wordpress.com/2019/06/06/new-tls-security-requirements-for-ios-13-and-macos-catalina-10-15/
[16]: https://superuser.com/questions/738612/openssl-ca-keyusage-extension/1248085#1248085
[17]: https://www.feistyduck.com/library/openssl-cookbook/online/ch-openssl.html
[18]: https://nginx.org/en/docs/http/server_names.html
