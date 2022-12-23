# 用户手册

## Certbot 命令

Certbot 使用许多不同的命令(也称为“子命令”)来请求特定的操作，如获取、更新或撤销证书。
本文档将讨论最重要和最常用的命令;一个详尽的列表也出现在文档的末尾。

如果您的系统使用较旧的包，web 服务器上的“certbot”脚本可能被命名为“letsencrypt”。
在整个文档中，无论何时看到“certbot”，请根据需要替换正确的名称。

## 获得证书(并选择插件)

Certbot 帮助你完成两个任务:

1. 获取证书:自动执行所需的身份验证步骤以证明您控制了域，将证书保存到“/etc/letsencrypt/live/”并定期更新。
2. 可选地，将该证书安装到受支持的 web 服务器(如 Apache 或 nginx)和其他类型的服务器。为了使用证书，可以自动修改服务器的配置。

要获取证书并安装它，请使用`certbot run`命令(或`certbot`，两者是相同的)。

如果只获取证书而不将其安装到任何地方，可以使用`certbot certonly` ("certificate only")命令。

!!! example "一些使用 Certbot 的例子"

    ```sh
    # 获取并安装证书:
    certbot

    # 获取证书但不安装:
    certbot certonly

    # 您可以使用-d指定多个域，并通过多次运行Certbot来获取和安装不同的证书:
    certbot certonly -d example.com -d www.example.com
    certbot certonly -d app.example.com -d api.example.com
    ```

要执行这些任务，Certbot 将要求您从身份验证器和安装程序插件中进行选择。
适当的插件选择将取决于您正在运行的服务器软件类型以及计划使用证书的服务器软件类型。

**身份验证器**是自动执行所需步骤以证明您控制了您试图请求证书的域名的插件。验证者总是需要获得证书。

**安装程序**是可以使用 Certbot 获得的证书自动修改您的 web 服务器配置以通过 HTTPS 服务您的网站的插件。
只有当您希望 Certbot 将证书安装到 web 服务器时，才需要安装程序。

有些插件同时是验证器和安装器，可以指定验证器和插件的不同[组合](#combination)。

| 插件                      | Auth | Inst | 解释                                                                                                                                                    | 挑战类型(和端口)                                |
| ------------------------- | ---- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| [apache](#apache)         | Y    | Y    | 使用 Apache 自动获取和安装证书。                                                                                                                        | [http-01][http-01] (80)                         |
| [nginx](#nginx)           | Y    | Y    | 使用 Nginx 自动获取和安装证书。                                                                                                                         | [http-01][http-01] (80)                         |
| [webroot](#webroot)       | Y    | N    | 通过写入已经运行的 web 服务器的 webroot 目录来获取证书。                                                                                                | [http-01][http-01] (80)                         |
| [standalone](#standalone) | Y    | N    | 使用"standalone" web 服务器获取证书。要求端口 80 可用。这在没有 web 服务器的系统中很有用，或者在不支持或不需要直接与本地 web 服务器集成的情况下很有用。 | [http-01][http-01] (80)                         |
| [dns plugs](#dns_plugins) | Y    | N    | 这类插件通过修改 DNS 记录来自动获取证书，以证明您拥有对域的控制权。以这种方式进行域验证是从 Let's Encrypt 获得通配符证书的唯一方法。                    | [dns-01](dns-01) (53)                           |
| [manual](#manual)         | Y    | N    | 根据说明手动获取证书，自行执行域验证。以这种方式创建的证书不支持自动更新。自动更新可以通过提供一个身份验证钩子脚本来实现域验证步骤的自动化。            | [http-01][http-01] (80) or [dns-01][dns-01](53) |

在底层，插件使用几个 ACME 协议挑战之一来证明您控制了一个域名。
选项是 [http-01][http-01](使用端口 80)和 [dns-01][dns-01](需要在端口 53 上配置 DNS 服务器，尽管这通常不是您的 web 服务器的同一台机器)。
一些插件支持多种挑战类型，在这种情况下，你可以选择一个带有`--preferred-challenges`的。

还有许多[第三方插件](#third-party-plugins)可用。
下面我们将更详细地描述每个插件可以使用的情况，以及如何使用它。

[challenges]: https://datatracker.ietf.org/doc/html/rfc8555#section-8
[http-01]: https://datatracker.ietf.org/doc/html/rfc8555#section-8.3
[dns-01]: https://datatracker.ietf.org/doc/html/rfc8555#section-8.4

### Apache

Apache 插件目前[支持](https://github.com/certbot/certbot/blob/master/certbot-apache/certbot_apache/_internal/entrypoint.py)基于 Debian、Fedora、SUSE、Gentoo、CentOS 和 Darwin 的现代操作系统。
这将自动在 Apache web 服务器上获取和安装证书。
要在命令行上指定这个插件，只需包含`--apache`。

### Webroot

如果你正在运行一个本地 web 服务器，你有能力修改所服务的内容，并且你不希望在证书颁发过程中停止 web 服务器，你可以使用 webroot 插件通过在命令行中包含`certonly` and `--webroot`来获取证书。
此外，你需要指定`--webroot-path` or `-w`与顶级目录("web root")，其中包含你的 web 服务器提供的文件。
例如，`--webroot-path /var/www/html` or `--webroot-path /usr/share/nginx/html`是两种常见的 webroot 路径。

如果您要同时获得多个域的证书，插件需要知道每个域的文件是从哪里提供的，这可能是每个域的单独目录。
当为多个域请求证书时，每个域将使用最近指定的`--webroot-path`。举个例子，

```sh
certbot certonly --webroot -w /var/www/example -d www.example.com -d example.com -w /var/www/other -d other.example.net -d another.other.example.net
```

would obtain a single certificate for all of those names, using the `/var/www/example` webroot directory for the first two, and `/var/www/other` for the second two.

The webroot plugin works by creating a temporary file for each of your requested domains in `${webroot-path}/.well-known/acme-challenge`.
Then the Let's Encrypt validation server makes HTTP requests to validate that the DNS for each requested domain resolves to the server running certbot.
An example request made to your web server would look like:

```
66.133.109.36 - - [05/Jan/2016:20:11:24 -0500] "GET /.well-known/acme-challenge/HGr8U1IeTW4kY_Z6UIyaakzOkyQgPr_7ArlLgtZE8SX HTTP/1.1" 200 87 "-" "Mozilla/5.0 (compatible; Let's Encrypt validation server; +https://www.letsencrypt.org)"
```

Note that to use the webroot plugin, your server must be configured to serve files from hidden directories.
If `/.well-known` is treated specially by your webserver configuration, you might need to modify the configuration to ensure that files inside `/.well-known/acme-challenge` are served by the webserver.

Under Windows, Certbot will generate a `web.config` file, if one does not already exist, in `/.well-known/acme-challenge` in order to let IIS serve the challenge files even if they do not have an extension.

### Nginx

Nginx 插件应该适用于大多数配置。
我们建议在使用 Nginx 之前备份它的配置(尽管你也可以使用`certbot --nginx rollback`将更改恢复到配置)。
你可以通过在命令行上提供`--nginx`标志来使用它。

```
certbot --nginx
```

<a id="standalone"></a>

### Standalone

如果不想使用(或目前没有)现有服务器软件，请使用独立模式获取证书。
独立插件不依赖于在您获取证书的机器上运行的任何其他服务器软件。

要使用“独立”web 服务器获取证书，您可以通过在命令行中包含`certonly`和`——standalone`来使用独立插件。
这个插件需要绑定到端口 80 来执行域验证，所以你可能需要停止你现有的 web 服务器。

您的机器必须仍然可以使用每个请求的域名在指定端口上接受来自 Internet 的入站连接。

默认情况下，Certbot 首先尝试使用 IPv6 绑定到所有接口的端口，然后使用 IPv4 绑定到该端口;只要至少有一个绑定成功，Certbot 就会继续。在大多数 Linux 系统上，IPv4 流量将被路由到绑定的 IPv6 端口，在第二次绑定期间失败是意料之中的。

使用`--<challenge-type>-address`显式告诉 Certbot 绑定哪个接口(和协议)。

<a id="dns_plugins"></a>

### DNS 插件

如果你想从 Let's Encrypt 获得一个通配符证书，或者在目标服务器以外的机器上运行“certbot”，你可以使用 certbot 的 DNS 插件之一。

这些插件不包含在默认的 Certbot 安装中，必须单独安装。
它们可以作为 Docker 映像和快照在许多 OS 包管理器中使用。
访问 <https://certbot.eff.org> 了解在您的系统上使用 DNS 插件的最佳方法。

安装完成后，你可以在以下地址找到如何使用每个插件的文档:

- [certbot-dns-cloudflare](https://certbot-dns-cloudflare.readthedocs.io)
- [certbot-dns-digitalocean](https://certbot-dns-digitalocean.readthedocs.io)
- [certbot-dns-dnsimple](https://certbot-dns-dnsimple.readthedocs.io)
- [certbot-dns-dnsmadeeasy](https://certbot-dns-dnsmadeeasy.readthedocs.io)
- [certbot-dns-gehirn](https://certbot-dns-gehirn.readthedocs.io)
- [certbot-dns-google](https://certbot-dns-google.readthedocs.io)
- [certbot-dns-linode](https://certbot-dns-linode.readthedocs.io)
- [certbot-dns-luadns](https://certbot-dns-luadns.readthedocs.io)
- [certbot-dns-nsone](https://certbot-dns-nsone.readthedocs.io)
- [certbot-dns-ovh](https://certbot-dns-ovh.readthedocs.io)
- [certbot-dns-rfc2136](https://certbot-dns-rfc2136.readthedocs.io)
- [certbot-dns-route53](https://certbot-dns-route53.readthedocs.io)
- [certbot-dns-sakuracloud](https://certbot-dns-sakuracloud.readthedocs.io)

<a id="manual"></a>

### 手动

如果你想在目标服务器以外的机器上获得运行`certbot`的证书，或者自己执行域验证的步骤，你可以使用手动插件。
虽然隐藏在 UI 中，但您可以使用插件通过在命令行上指定`certonly` 和 `--manual`来获取证书。
这要求您将命令复制并粘贴到另一个终端会话中，该会话可能位于不同的计算机上。

手动插件可以使用`http` or `dns`挑战。您可以使用`--preferred-challenges`选项来选择您喜欢的挑战。

`http` 挑战将要求您将具有特定名称和特定内容的文件直接放置在顶级目录(“web root”)的目录中，该目录包含您的 web 服务器提供的文件。
本质上它和[webroot](#webroot)插件是一样的，但不是自动化的。

当使用`dns`挑战时，`certbot`会要求你在域名下放置一个包含特定内容的 TXT DNS 记录，该记录由你想要颁发证书的主机名组成，以`_acme-challenge`为前缀。

!!! example "对于域名`example.com`，区域文件条目看起来是这样的"

    _acme-challenge.example.com. 300 IN TXT "gfj9Xq...Rg85nM"

<a id="manual-renewal"></a>

**使用手动插件更新**

使用`--manual` 创建的证书**不**支持自动更新，除非通过`--manual-auth-hook`与[认证钩子脚本](#hooks)结合使用，以自动设置所需的 HTTP 和/或 TXT 挑战。

如果您可以使用其他支持自动更新的[插件](#plugins)来创建证书，强烈建议您这样做。

如果要手动更新不带钩子的`--manual`证书，请重复最初创建证书时使用的`certbot --manual`命令。
由于这需要复制和粘贴新的 HTTP 文件或 DNS TXT 记录，因此 cron 作业不能自动执行该命令。

<a id="combination"></a>

### 组合插件

有时您可能希望指定不同的验证程序和安装程序插件的组合。
为此，使用`--authenticator` 或 `-a` 指定验证程序插件，使用`--installer` 或 `-i`指定安装程序插件。

例如，您可以使用[webroot](#webroot)插件进行身份验证，使用[apache](#apache)插件进行安装来创建证书。

```sh
certbot run -a webroot -i apache -w /var/www/html -d example.com
```

或者你可以使用手动插件进行身份验证，使用[nginx](nginx)插件进行安装来创建证书。(注意此证书不能自动更新。)

```sh
certbot run -a manual -i nginx -d example.com
```

<a id="third-party-plugins"></a>

### 第三方插件

还有许多第三方插件，由其他开发人员提供。许多是测试版/试验性的，但有些已经被广泛使用:

| Plugin            | Auth | Inst | Notes                                                         |
| ----------------- | ---- | ---- | ------------------------------------------------------------- |
| [haproxy]         | Y    | Y    | Integration with the HAProxy load balancer                    |
| [s3front]         | Y    | Y    | Integration with Amazon CloudFront distribution of S3 buckets |
| [gandi]           | Y    | N    | Obtain certificates via the Gandi LiveDNS API                 |
| [varnish]         | Y    | N    | Obtain certificates via a Varnish server                      |
| [external-auth]   | Y    | Y    | A plugin for convenient scripting                             |
| [pritunl]         | N    | Y    | Install certificates in pritunl distributed OpenVPN servers   |
| [proxmox]         | N    | Y    | Install certificates in Proxmox Virtualization servers        |
| [dns-standalone]  | Y    | N    | Obtain certificates via an integrated DNS server              |
| [dns-ispconfig]   | Y    | N    | DNS Authentication using ISPConfig as DNS server              |
| [dns-clouddns]    | Y    | N    | DNS Authentication using CloudDNS API                         |
| [dns-lightsail]   | Y    | N    | DNS Authentication using Amazon Lightsail DNS API             |
| [dns-inwx]        | Y    | Y    | DNS Authentication for INWX through the XML API               |
| [dns-azure]       | Y    | N    | DNS Authentication using Azure DNS                            |
| [dns-godaddy]     | Y    | N    | DNS Authentication using Godaddy DNS                          |
| [dns-yandexcloud] | Y    | N    | DNS Authentication using Yandex Cloud DNS                     |
| [dns-bunny]       | Y    | N    | DNS Authentication using BunnyDNS                             |
| [njalla]          | Y    | N    | DNS Authentication for njalla                                 |
| [DuckDNS]         | Y    | N    | DNS Authentication for DuckDNS                                |
| [Porkbun]         | Y    | N    | DNS Authentication for Porkbun                                |
| [Infomaniak]      | Y    | N    | DNS Authentication using Infomaniak Domains API               |
| [dns-multi]       | Y    | N    | DNS authentication of 100+ providers using go-acme/lego       |

[haproxy]: https://github.com/greenhost/certbot-haproxy
[s3front]: https://github.com/dlapiduz/letsencrypt-s3front
[gandi]: https://github.com/obynio/certbot-plugin-gandi
[varnish]: https://git.sesse.net/?p=letsencrypt-varnish-plugin
[pritunl]: https://github.com/kharkevich/letsencrypt-pritunl
[proxmox]: https://github.com/kharkevich/letsencrypt-proxmox
[external-auth]: https://github.com/EnigmaBridge/certbot-external-auth
[dns-standalone]: https://github.com/siilike/certbot-dns-standalone
[dns-ispconfig]: https://github.com/m42e/certbot-dns-ispconfig
[dns-clouddns]: https://github.com/vshosting/certbot-dns-clouddns
[dns-lightsail]: https://github.com/noi/certbot-dns-lightsail
[dns-inwx]: https://github.com/oGGy990/certbot-dns-inwx/
[dns-azure]: https://github.com/binkhq/certbot-dns-azure
[dns-godaddy]: https://github.com/miigotu/certbot-dns-godaddy
[dns-yandexcloud]: https://github.com/PykupeJIbc/certbot-dns-yandexcloud
[dns-bunny]: https://github.com/mwt/certbot-dns-bunny
[njalla]: https://github.com/chaptergy/certbot-dns-njalla
[duckdns]: https://github.com/infinityofspace/certbot_dns_duckdns
[porkbun]: https://github.com/infinityofspace/certbot_dns_porkbun
[infomaniak]: https://github.com/Infomaniak/certbot-dns-infomaniak
[dns-multi]: https://github.com/alexzorin/certbot-dns-multi

如果你感兴趣，你也可以[编写自己的插件](#dev-plugin).

<a id="managing-certs"></a>

## 管理证书

要查看 Certbot 知道的证书列表，运行`certificates`子命令:

```sh
certbot certificates
```

This returns information in the following format::

```
Found the following certificates:
Certificate Name: example.com
Domains: example.com, www.example.com
Expiry Date: 2017-02-19 19:53:00+00:00 (VALID: 30 days)
Certificate Path: /etc/letsencrypt/live/example.com/fullchain.pem
Key Type: RSA
Private Key Path: /etc/letsencrypt/live/example.com/privkey.pem
```

`Certificate Name` shows the name of the certificate.
Pass this name using the `--cert-name` flag to specify a particular certificate for the `run`, `certonly`, `certificates`, `renew`, and `delete` commands. Example::

```sh
certbot certonly --cert-name example.com
```

<a id="updating_certs"></a>

### 重新创建和更新已有证书

You can use `certonly` or `run` subcommands to request
the creation of a single new certificate even if you already have an
existing certificate with some of the same domain names.

If a certificate is requested with `run` or `certonly` specifying a
certificate name that already exists, Certbot updates
the existing certificate. Otherwise a new certificate
is created and assigned the specified name.

The `--force-renewal`, `--duplicate`, and `--expand` options
control Certbot's behavior when re-creating
a certificate with the same name as an existing certificate.
If you don't specify a requested behavior, Certbot may ask you what you intended.

`--force-renewal` tells Certbot to request a new certificate
with the same domains as an existing certificate. Each domain
must be explicitly specified via `-d`. If successful, this certificate
is saved alongside the earlier one and symbolic links (the "`live`"
reference) will be updated to point to the new certificate. This is a
valid method of renewing a specific individual
certificate.

`--duplicate` tells Certbot to create a separate, unrelated certificate
with the same domains as an existing certificate. This certificate is
saved completely separately from the prior one. Most users will not
need to issue this command in normal circumstances.

`--expand` tells Certbot to update an existing certificate with a new
certificate that contains all of the old domains and one or more additional
new domains. With the `--expand` option, use the `-d` option to specify
all existing domains and one or more new domains.

Example:

```sh
certbot --expand -d existing.com,example.com,newdomain.com
```

If you prefer, you can specify the domains individually like this:

```sh
certbot --expand -d existing.com -d example.com -d newdomain.com
```

Consider using `--cert-name` instead of `--expand`, as it gives more control
over which certificate is modified and it lets you remove domains as well as adding them.

`--allow-subset-of-names` tells Certbot to continue with certificate generation if
only some of the specified domain authorizations can be obtained. This may
be useful if some domains specified in a certificate no longer point at this
system.

Whenever you obtain a new certificate in any of these ways, the new
certificate exists alongside any previously obtained certificates, whether
or not the previous certificates have expired. The generation of a new
certificate counts against several rate limits that are intended to prevent
abuse of the ACME protocol, as described [here](https://letsencrypt.org/docs/rate-limits/).

<a id="changing"></a>

### 更改证书的域

The `--cert-name` flag can also be used to modify the domains a certificate contains,
by specifying new domains using the `-d` or `--domains` flag. If certificate `example.com`
previously contained `example.com` and `www.example.com`, it can be modified to only
contain `example.com` by specifying only `example.com` with the `-d` or `--domains` flag. Example::

```sh
certbot certonly --cert-name example.com -d example.com
```

The same format can be used to expand the set of domains a certificate contains, or to replace that set entirely::

```sh
certbot certonly --cert-name example.com -d example.org,www.example.org
```

<a id="using-ecdsa-keys"></a>

### RSA 和 ECDSA 钥匙

Certbot supports two certificate private key algorithms: `rsa` and `ecdsa`.

As of version 2.0.0, Certbot defaults to ECDSA `secp256r1` (P-256) certificate private keys
for all new certificates. Existing certificates will continue to renew using their existing key
type, unless a key type change is requested.

The type of key used by Certbot can be controlled through the `--key-type` option.
You can use the `--elliptic-curve` option to control the curve used in ECDSA
certificates and the `--rsa-key-size` option to control the size of RSA keys.

.. warning:: If you obtain certificates using ECDSA keys, you should be careful
not to downgrade to a Certbot version earlier than 1.10.0 where ECDSA keys were
not supported. Downgrades like this are possible if you switch from something like
the snaps or pip to packages provided by your operating system which often lag behind.

Changing a certificate's key type

````

Unless you are aware that you need to support very old HTTPS clients that are
not supported by most sites, you can safely transition your site to use
ECDSA keys instead of RSA keys.

If you want to change a single certificate to use ECDSA keys, you'll need to
create or renew a certificate while setting `--key-type ecdsa` on the command line:

```shell
  certbot renew --key-type ecdsa --cert-name example.com --force-renewal
```

If you want to use ECDSA keys for all certificates in the future (including renewals
of existing certificates), you can add the following line to Certbot's
:ref:`configuration file <config-file>`:

``` ini
  key-type = ecdsa
````

which will take effect upon the next renewal of each certificate.

## 撤销证书

If you need to revoke a certificate, use the `revoke` subcommand to do so.

A certificate may be revoked by providing its name (see `certbot certificates`) or by providing
its path directly

```sh
  certbot revoke --cert-name example.com
  certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem
```

If the certificate being revoked was obtained via the `--staging`, `--test-cert` or a non-default `--server` flag,
that flag must be passed to the `revoke` subcommand.

!!! note

    After revocation, Certbot will (by default) ask whether you want to **delete** the certificate.
    Unless deleted, Certbot will try to renew revoked certificates the next time ``certbot renew`` runs.

You can also specify the reason for revoking your certificate by using the `reason` flag.
Reasons include `unspecified` which is the default, as well as `keycompromise`, `affiliationchanged`, `superseded`, and `cessationofoperation`

```sh
certbot revoke --cert-name example.com --reason keycompromise
```

Revoking by account key or certificate private key

By default, Certbot will try revoke the certificate using your ACME account key. If the certificate was created from the same ACME account, the revocation will be successful.

If you instead have the corresponding private key file to the certificate you wish to revoke, use `--key-path` to perform the revocation from any ACME account

```sh
certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem --key-path /etc/letsencrypt/live/example.com/privkey.pem
```

<a id="deleting"></a>

### 删除证书

If you need to delete a certificate, use the `delete` subcommand.

!!! note

    Read this and the `Safely deleting certificates`\_ sections carefully. This is an irreversible operation and must be done with care.

Certbot does not automatically revoke a certificate before deleting it. If you're no longer using a certificate and don't
plan to use it anywhere else, you may want to follow the instructions in `Revoking certificates`\_ instead. Generally, there's
no need to revoke a certificate if its private key has not been compromised, but you may still receive expiration emails
from Let's Encrypt unless you revoke.

!!! note

    Do not manually delete certificate files from inside `/etc/letsencrypt/`. Always use the `delete` subcommand.

A certificate may be deleted by providing its name with `--cert-name`. \
You may find its name using `certbot certificates`.

Otherwise, you will be prompted to choose one or more certificates to delete

```sh
certbot delete --cert-name example.com

## or to choose from a list:

certbot delete

Safely deleting certificates

```

Deleting a certificate without following the proper steps can result in a non-functioning server. To safely delete a
certificate, follow all the steps below to make sure that references to a certificate are removed from the configuration
of any installed server software (Apache, nginx, Postfix, etc) _before_ deleting the certificate.

To explain further, when installing a certificate, Certbot modifies Apache or nginx's configuration to load the certificate
and its private key from the `/etc/letsencrypt/live/` directory. Before deleting a certificate, it is necessary to undo
that modification, by removing any references to the certificate from the webserver's configuration files.

Follow these steps to safely delete a certificate:

1. Find all references to the certificate (substitute `example.com` in the command for the name of the certificate
   you wish to delete)

   ```sh
    sudo bash -c 'grep -R live/example.com /etc/{nginx,httpd,apache2}'
   ```

   If there are no references found, skip directly to Step 4.

   If some references are found, they will look something like

   ```sh
   /etc/apache2/sites-available/000-default-le-ssl.conf:SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
   /etc/apache2/sites-available/000-default-le-ssl.conf:SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
   ```

2. You will need a self-signed certificate to replace the certificate you are deleting. The following command will generate one
   for you, saving the certificate at `/etc/letsencrypt/self-signed-cert.pem` and its private key at `/etc/letsencrypt/self-signed-privkey.pem`
   ```sh
    sudo openssl req -nodes -batch -x509 -newkey rsa:2048 -keyout /etc/letsencrypt/self-signed-privkey.pem -out /etc/letsencrypt/self-signed-cert.pem -days 356
   ```
3. For each reference found in Step 1, open the file in a text editor and replace the reference to the existing
   certificate with a reference to the self-signed certificate.

   Continuing from the previous example, you would open `/etc/apache2/sites-available/000-default-le-ssl.conf` in a text editor
   and modify the two matching lines of text to instead say

   ```sh
    SSLCertificateFile /etc/letsencrypt/self-signed-cert.pem
    SSLCertificateKeyFile /etc/letsencrypt/self-signed-privkey.pem
   ```

4. It is now safe to delete the certificate. Do so by running

   ```sh
    sudo certbot delete --cert-name example.com
   ```

<a id="renewal"></a>

## 更新的证书

!!! note

    Let's Encrypt CA issues short-lived certificates (90 days). Make sure you renew the certificates at least once in 3 months.

!!! quote

    Most Certbot installations come with automatic renewal out of the box. See `Automated Renewals`\_ for more details.

!!! quote

    Users of the `Manual`_ plugin should note that `--manual` certificates will not renew automatically, unless combined with authentication hook scripts.
    See `Renewal with the manual plugin <#manual-renewal>`_.

As of version 0.10.0, Certbot supports a `renew` action to check all installed certificates for impending expiry and attempt to renew them.
The simplest form is simply

```sh
certbot renew
```

This command attempts to renew any previously-obtained certificates that expire in less than 30 days.
The same plugin and options that were used at the time the certificate was originally issued will be used for the
renewal attempt, unless you specify other plugins or options. Unlike `certonly`, `renew` acts on
multiple certificates and always takes into account whether each one is near
expiry. Because of this, `renew` is suitable (and designed) for automated use,
to allow your system to automatically renew each certificate when appropriate.
Since `renew` only renews certificates that are near expiry it can be
run as frequently as you want - since it will usually take no action.

The `renew` command includes hooks for running commands or scripts before or after a certificate is
renewed. For example, if you have a single certificate obtained using
the standalone\_ plugin, you might need to stop the webserver
before renewing so standalone can bind to the necessary ports, and
then restart it after the plugin is finished. Example::

certbot renew --pre-hook "service nginx stop" --post-hook "service nginx start"

If a hook exits with a non-zero exit code, the error will be printed
to `stderr` but renewal will be attempted anyway. A failing hook
doesn't directly cause Certbot to exit with a non-zero exit code, but
since Certbot exits with a non-zero exit code when renewals fail, a
failed hook causing renewal failures will indirectly result in a
non-zero exit code. Hooks will only be run if a certificate is due for
renewal, so you can run the above command frequently without
unnecessarily stopping your webserver.

When Certbot detects that a certificate is due for renewal, `--pre-hook`
and `--post-hook` hooks run before and after each attempt to renew it.
If you want your hook to run only after a successful renewal, use
`--deploy-hook` in a command like this.

`certbot renew --deploy-hook /path/to/deploy-hook-script`

You can also specify hooks by placing files in subdirectories of Certbot's
configuration directory. Assuming your configuration directory is
`/etc/letsencrypt`, any executable files found in
`/etc/letsencrypt/renewal-hooks/pre`,
`/etc/letsencrypt/renewal-hooks/deploy`, and
`/etc/letsencrypt/renewal-hooks/post` will be run as pre, deploy, and post
hooks respectively when any certificate is renewed with the `renew`
subcommand. These hooks are run in alphabetical order and are not run for other
subcommands. (The order the hooks are run is determined by the byte value of
the characters in their filenames and is not dependent on your locale.)

Hooks specified in the command line, :ref:`configuration file
<config-file>`, or :ref:`renewal configuration files <renewal-config-file>` are
run as usual after running all hooks in these directories. One minor exception
to this is if a hook specified elsewhere is simply the path to an executable
file in the hook directory of the same type (e.g. your pre-hook is the path to
an executable in `/etc/letsencrypt/renewal-hooks/pre`), the file is not run a
second time. You can stop Certbot from automatically running executables found
in these directories by including `--no-directory-hooks` on the command line.

More information about hooks can be found by running
`certbot --help renew`.

If you're sure that this command executes successfully without human
intervention, you can add the command to `crontab` (since certificates
are only renewed when they're determined to be near expiry, the command
can run on a regular basis, like every week or every day). In that case,
you are likely to want to use the `-q` or `--quiet` quiet flag to
silence all output except errors.

If you are manually renewing all of your certificates, the
`--force-renewal` flag may be helpful; it causes the expiration time of
the certificate(s) to be ignored when considering renewal, and attempts to
renew each and every installed certificate regardless of its age. (This
form is not appropriate to run daily because each certificate will be
renewed every day, which will quickly run into the certificate authority
rate limit.)

Note that options provided to `certbot renew` will apply to
_every_ certificate for which renewal is attempted; for example,
`certbot renew --rsa-key-size 4096` would try to replace every
near-expiry certificate with an equivalent certificate using a 4096-bit
RSA public key. If a certificate is successfully renewed using
specified options, those options will be saved and used for future
renewals of that certificate.

An alternative form that provides for more fine-grained control over the
renewal process (while renewing specified certificates one at a time),
is `certbot certonly` with the complete set of subject domains of
a specific certificate specified via `-d` flags. You may also want to
include the `-n` or `--noninteractive` flag to prevent blocking on
user input (which is useful when running the command from cron).

`certbot certonly -n -d example.com -d www.example.com`

All of the domains covered by the certificate must be specified in
this case in order to renew and replace the old certificate rather
than obtaining a new one; don't forget any `www.` domains! Specifying
a subset of the domains creates a new, separate certificate containing
only those domains, rather than replacing the original certificate.
When run with a set of domains corresponding to an existing certificate,
the `certonly` command attempts to renew that specific certificate.

Please note that the CA will send notification emails to the address
you provide if you do not renew certificates that are about to expire.

Certbot is working hard to improve the renewal process, and we
apologize for any inconvenience you encounter in integrating these
commands into your individual environment.

!!! note

    `certbot renew` exit status will only be 1 if a renewal attempt failed.

This means `certbot renew` exit status will be 0 if no certificate needs to be updated.
If you write a custom script and expect to run a command only after a certificate was actually renewed
you will need to use the `--deploy-hook` since the exit status will be 0 both on successful renewal
and when renewal is not necessary.

.. \_renewal-config-file:
.. \_Modifying the Renewal Configuration File:

## 修改已有证书的更新配置

When creating a certificate, Certbot will keep track of all of the relevant options chosen by the user. At renewal
time, Certbot will remember these options and apply them once again.

Sometimes, you may encounter the need to change some of these options for future certificate renewals. To achieve this,
you will need to perform the following steps:

1. Perform a _dry run renewal_ with the amended options on the command line. This allows you to confirm that the change
   is valid and will result in successful future renewals.
2. If the dry run is successful, perform a _live renewal_ of the certificate. This will persist the change for future
   renewals. If the certificate is not yet due to expire, you will need to force a renewal using `--force-renewal`.

!!! note

    Rate limits from the certificate authority may prevent you from performing multiple renewals in a short

period of time. It is strongly recommended to perform the second step only once, when you have decided on what
options should change.

As a practical example, if you were using the `webroot` authenticator and had relocated your website to another directory,
you would need to change the `--webroot-path` to the new directory. Following the above advice:

1. Perform a _dry-run renewal_ of the individual certificate with the amended options::
   ```sh
    certbot renew --cert-name example.com --webroot-path /path/to/new/location --dry-run
   ```
2. If the dry-run was successful, make the change permanent by performing a _live renewal_ of the certificate with the
   amended options, including `--force-renewal`::

   ```sh
    certbot renew --cert-name example.com --webroot-path /path/to/new/location --force-renewal
   ```

   `--cert-name` selects the particular certificate to be modified. Without this option, all certificates will be selected.

   `--webroot-path` is the option intended to be changed. All other previously selected options will be kept the same
   and do not need to be included in the command.

For advanced certificate management tasks, it is also possible to manually modify the certificate's renewal configuration
file, but this is discouraged since it can easily break Certbot's ability to renew your certificates. These renewal
configuration files are located at `/etc/letsencrypt/renewal/CERTNAME.conf`. If you choose to modify the renewal
configuration file we advise you to make a backup of the file beforehand and test its validity with the `certbot renew --dry-run` command.

.. warning:: Manually modifying files under `/etc/letsencrypt/renewal/` can damage them if done improperly and we do not recommend doing so.

## 自动更新

Most Certbot installations come with automatic renewals preconfigured. This
is done by means of a scheduled task which runs `certbot renew` periodically.

If you are unsure whether you need to configure automated renewal:

1. Review the instructions for your system and installation method at
   https://certbot.eff.org/instructions. They will describe how to set up a scheduled task,
   if necessary. If no step is listed, your system comes with automated renewal pre-installed,
   and you should not need to take any additional actions.
2. On Linux and BSD, you can check to see if your installation method has pre-installed a timer
   for you. To do so, look for the `certbot renew` command in either your system's crontab
   (typically `/etc/crontab` or `/etc/cron.*/*`) or systemd timers (`systemctl list-timers`).
3. If you're still not sure, you can configure automated renewal manually by following the steps
   in the next section. Certbot has been carefully engineered to handle the case where both manual
   automated renewal and pre-installed automated renewal are set up.

Setting up automated renewal

If you think you may need to set up automated renewal, follow these instructions to set up a
scheduled task to automatically renew your certificates in the background. If you are unsure
whether your system has a pre-installed scheduled task for Certbot, it is safe to follow these
instructions to create one.

!!! note

    If you're using Windows, these instructions are not neccessary as Certbot on Windows comes with a scheduled task for automated renewal pre-installed.

If you are using macOS and installed Certbot using Homebrew, follow the instructions at
https://certbot.eff.org/instructions to set up automated renewal. The instructions below
are not applicable on macOS.

Run the following line, which will add a cron job to `/etc/crontab`:

```shell
SLEEPTIME=$(awk 'BEGIN{srand(); print int(rand()_(3600+1))}'); echo "0 0,12 _ \* \* root sleep $SLEEPTIME && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

If you needed to stop your webserver to run Certbot, you'll want to
add `pre` and `post` hooks to stop and start your webserver automatically.
For example, if your webserver is HAProxy, run the following commands to create the hook files
in the appropriate directory:

```shell
sudo sh -c 'printf "#!/bin/sh\nservice haproxy stop\n" > /etc/letsencrypt/renewal-hooks/pre/haproxy.sh'
sudo sh -c 'printf "#!/bin/sh\nservice haproxy start\n" > /etc/letsencrypt/renewal-hooks/post/haproxy.sh'
sudo chmod 755 /etc/letsencrypt/renewal-hooks/pre/haproxy.sh
sudo chmod 755 /etc/letsencrypt/renewal-hooks/post/haproxy.sh
```

Congratulations, Certbot will now automatically renew your certificates in the background.

If you are interested in learning more about how Certbot renews your certificates, see the
`Renewing certificates`\_ section above.

<a id="where-certs"></a>

## 我的证书在哪里?

所有生成的密钥和颁发的证书都可以在`/etc/letsencrypt/live/$domain`中找到，其中`$domain`是证书名称(参见下面的说明)。
不要复制，请将您的(web)服务器配置直接指向这些文件(或创建符号链接)。
在更新期间，`/etc/letsencrypt/live`将使用最新的必要文件进行更新。

!!! note

    在路径`/etc/letsencrypt/live/$domain`中使用的证书名称`$domain`遵循以下约定:

- 这是它的名字`--cert-name`,
- 如果用户没有设置`--cert-name`，它是给`--domains`的第一个域，
- 如果第一个域是通配符域(例如;`*.example.com`)证书名称将为`example.com`，
- 如果与已命名为`example.com`的证书发生名称冲突，则将使用`example.com-001`数字序列构造新的证书名称。.

For historical reasons, the containing directories are created with
permissions of `0700` meaning that certificates are accessible only
to servers that run as the root user. **If you will never downgrade
to an older version of Certbot**, then you can safely fix this using
`chmod 0755 /etc/letsencrypt/{live,archive}`.

For servers that drop root privileges before attempting to read the
private key file, you will also need to use `chgrp` and `chmod
0640` to allow the server to read
`/etc/letsencrypt/live/$domain/privkey.pem`.

!!! note

    `/etc/letsencrypt/archive` and `/etc/letsencrypt/keys`

contain all previous keys and certificates, while `/etc/letsencrypt/live` symlinks to the latest versions.

The following files are available:

`privkey.pem` Private key for the certificate.

!!! warning

    This **must be kept secret at all times**! Never share it with anyone, including Certbot developers.
    You cannot put it into a safe, however - your server still needs to access this file in order for SSL/TLS to work.

!!! note

    As of Certbot version 0.29.0, private keys for new certificate default to `0600`.
    Any changes to the group mode or group owner (gid) of this file will be preserved on renewals.

This is what Apache needs for `SSLCertificateKeyFile  <https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatekeyfile>`_,
and Nginx for `ssl_certificate_key  <https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key>`_.

`fullchain.pem`
All certificates, **including** server certificate (aka leaf certificate or
end-entity certificate). The server certificate is the first one in this file,
followed by any intermediates.

This is what Apache >= 2.4.8 needs for `SSLCertificateFile  <https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatefile>`_,
and what Nginx needs for `ssl_certificate  <https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate>`_.

`cert.pem` and `chain.pem` (less common)
`cert.pem` contains the server certificate by itself, and
`chain.pem` contains the additional intermediate certificate or
certificates that web browsers will need in order to validate the
server certificate. If you provide one of these files to your web
server, you **must** provide both of them, or some browsers will show
"This Connection is Untrusted" errors for your site, `some of the time
  <https://whatsmychaincert.com/>`\_.

Apache < 2.4.8 needs these for `SSLCertificateFile  <https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatefile>`_.
and `SSLCertificateChainFile  <https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatechainfile>`_,
respectively.

If you're using OCSP stapling with Nginx >= 1.3.7, `chain.pem` should be
provided as the `ssl_trusted_certificate  <https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_trusted_certificate>`\_
to validate OCSP responses.

!!! note

    All files are PEM-encoded.

If you need other format, such as DER or PFX, then you
could convert using `openssl`. You can automate that with
`--deploy-hook` if you're using automatic renewal\_.

<a id="hooks"></a>

## 前和后验证挂钩

Certbot allows for the specification of pre and post validation hooks when run
in manual mode. The flags to specify these scripts are `--manual-auth-hook`
and `--manual-cleanup-hook` respectively and can be used as follows:

```sh
certbot certonly --manual --manual-auth-hook /path/to/http/authenticator.sh --manual-cleanup-hook /path/to/http/cleanup.sh -d secure.example.com
```

This will run the `authenticator.sh` script, attempt the validation, and then run
the `cleanup.sh` script. Additionally certbot will pass relevant environment
variables to these scripts:

- `CERTBOT_DOMAIN`: The domain being authenticated
- `CERTBOT_VALIDATION`: The validation string
- `CERTBOT_TOKEN`: Resource name part of the HTTP-01 challenge (HTTP-01 only)
- `CERTBOT_REMAINING_CHALLENGES`: Number of challenges remaining after the current challenge
- `CERTBOT_ALL_DOMAINS`: A comma-separated list of all domains challenged for the current certificate

Additionally for cleanup:

- `CERTBOT_AUTH_OUTPUT`: Whatever the auth script wrote to stdout

Example usage for HTTP-01:

```sh
certbot certonly --manual --preferred-challenges=http --manual-auth-hook /path/to/http/authenticator.sh --manual-cleanup-hook /path/to/http/cleanup.sh -d secure.example.com
```

```sh title="/path/to/http/authenticator.sh"
#!/bin/bash
echo $CERTBOT_VALIDATION > /var/www/htdocs/.well-known/acme-challenge/$CERTBOT_TOKEN
```

```sh title="/path/to/http/cleanup.sh"
#!/bin/bash
rm -f /var/www/htdocs/.well-known/acme-challenge/$CERTBOT_TOKEN
```

!!! example "DNS-01 (Cloudflare API v4)的使用示例(仅供示例使用，不按原样使用)"

    ```sh
    certbot certonly --manual --preferred-challenges=dns --manual-auth-hook /path/to/dns/authenticator.sh --manual-cleanup-hook /path/to/dns/cleanup.sh -d secure.example.com
    ```

```sh title="/path/to/dns/authenticator.sh"
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="your.email@example.com"

# Strip only the top domain to get the zone id
DOMAIN=$(expr match "$CERTBOT_DOMAIN" '.*\.\(.*\..*\)')

# Get the Cloudflare zone id
ZONE_EXTRA_PARAMS="status=active&page=1&per_page=20&order=status&direction=desc&match=all"
ZONE_ID=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=$DOMAIN&$ZONE_EXTRA_PARAMS" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" | python -c "import sys,json;print(json.load(sys.stdin)['result'][0]['id'])")

# Create TXT record
CREATE_DOMAIN="_acme-challenge.$CERTBOT_DOMAIN"
RECORD_ID=$(curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" \
     --data '{"type":"TXT","name":"'"$CREATE_DOMAIN"'","content":"'"$CERTBOT_VALIDATION"'","ttl":120}' \
             | python -c "import sys,json;print(json.load(sys.stdin)['result']['id'])")
# Save info for cleanup
if [ ! -d /tmp/CERTBOT_$CERTBOT_DOMAIN ];then
        mkdir -m 0700 /tmp/CERTBOT_$CERTBOT_DOMAIN
fi
echo $ZONE_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
echo $RECORD_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID

# Sleep to make sure the change has time to propagate over to DNS
sleep 25
```

```sh title="/path/to/dns/cleanup.sh"
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="your.email@example.com"

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID ]; then
        ZONE_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
fi

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID ]; then
        RECORD_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID
fi

# Remove the challenge TXT record from the zone
if [ -n "${ZONE_ID}" ]; then
    if [ -n "${RECORD_ID}" ]; then
        curl -s -X DELETE "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
                -H "X-Auth-Email: $EMAIL" \
                -H "X-Auth-Key: $API_KEY" \
                -H "Content-Type: application/json"
    fi
fi
```

<a id="lock-files"></a>

## 更改 ACME 服务器

By default, Certbot uses Let's Encrypt's production server at https://acme-v02.api.letsencrypt.org/directory.
You can tell Certbot to use a different CA by providing `--server` on the command line or in a :ref:`configuration file <config-file>` with the URL of the server's
ACME directory. For example, if you would like to use Let's Encrypt's
staging server, you would add `--server
https://acme-staging-v02.api.letsencrypt.org/directory` to the command line.

If Certbot does not trust the SSL certificate used by the ACME server, you
can use the `REQUESTS_CA_BUNDLE
<https://requests.readthedocs.io/en/latest/user/advanced/#ssl-cert-verification>`\_
environment variable to override the root certificates trusted by Certbot. Certbot
uses the `requests` library, which does not use the operating system trusted root store.

If you use `--server` to specify an ACME CA that implements the standardized
version of the spec, you may be able to obtain a certificate for a
wildcard domain. Some CAs (such as Let's Encrypt) require that domain
validation for wildcard domains must be done through modifications to
DNS records which means that the dns-01* challenge type must be used. To
see a list of Certbot plugins that support this challenge type and how
to use them, see plugins*.

## 锁文件

When processing a validation Certbot writes a number of lock files on your system
to prevent multiple instances from overwriting each other's changes. This means
that by default two instances of Certbot will not be able to run in parallel.

Since the directories used by Certbot are configurable, Certbot
will write a lock file for all of the directories it uses. This include Certbot's
`--work-dir`, `--logs-dir`, and `--config-dir`. By default these are
`/var/lib/letsencrypt`, `/var/log/letsencrypt`, and `/etc/letsencrypt`
respectively. Additionally if you are using Certbot with Apache or nginx it will
lock the configuration folder for that program, which are typically also in the
`/etc` directory.

Note that these lock files will only prevent other instances of Certbot from
using those directories, not other processes. If you'd like to run multiple
instances of Certbot simultaneously you should specify different directories
as the `--work-dir`, `--logs-dir`, and `--config-dir` for each instance
of Certbot that you would like to run.

<a id="config-file"></a>

## 配置文件

Certbot accepts a global configuration file that applies its options to all invocations
of Certbot. Certificate specific configuration choices should be set in the `.conf`
files that can be found in `/etc/letsencrypt/renewal`.

By default no cli.ini file is created (though it may exist already if you installed Certbot
via a package manager, for instance).
After creating one it is possible to specify the location of this configuration file with
`certbot --config cli.ini` (or shorter `-c cli.ini`). An
example configuration file is shown below:

```sh
# This is an example of the kind of things you can do in a configuration file.
# All flags used by the client can be configured here. Run Certbot with
# "--help" to learn more about the available options.
#
# Note that these options apply automatically to all use of Certbot for
# obtaining or renewing certificates, so options specific to a single
# certificate on a system with several certificates should not be placed
# here.

# Use ECC for the private key
key-type = ecdsa
elliptic-curve = secp384r1

# Use a 4096 bit RSA key instead of 2048
rsa-key-size = 4096

# Uncomment and update to register with the specified e-mail address
# email = foo@example.com

# Uncomment to use the standalone authenticator on port 443
# authenticator = standalone

# Uncomment to use the webroot authenticator. Replace webroot-path with the
# path to the public_html / webroot folder being served by your web server.
# authenticator = webroot
# webroot-path = /usr/share/nginx/html

# Uncomment to automatically agree to the terms of service of the ACME server
# agree-tos = true

# An example of using an alternate ACME server that uses EAB credentials
# server = https://acme.sectigo.com/v2/InCommonRSAOV
# eab-kid = somestringofstuffwithoutquotes
# eab-hmac-key = yaddayaddahexhexnotquoted
```

By default, the following locations are searched:

- `/etc/letsencrypt/cli.ini`
- `$XDG_CONFIG_HOME/letsencrypt/cli.ini` (or `~/.config/letsencrypt/cli.ini` if `$XDG_CONFIG_HOME` is not set).

Since this configuration file applies to all invocations of certbot it is incorrect
to list domains in it. Listing domains in cli.ini may prevent renewal from working.
Additionally due to how arguments in cli.ini are parsed, options which wish to
not be set should not be listed. Options set to false will instead be read
as being set to true by older versions of Certbot, since they have been listed
in the config file.

.. keep it up to date with constants.py

<a id="log-rotation"></a>

## 日志轮转

By default certbot stores status logs in `/var/log/letsencrypt`. By default
certbot will begin rotating logs once there are 1000 logs in the log directory.
Meaning that once 1000 files are in `/var/log/letsencrypt` Certbot will delete
the oldest one to make room for new logs. The number of subsequent logs can be
changed by passing the desired number to the command line flag
`--max-log-backups`. Setting this flag to 0 disables log rotation entirely,
causing certbot to always append to the same log file.

!!! note

    Some distributions, including Debian and Ubuntu, disable certbot's internal log rotation in favor of a more traditional logrotate script.
    If you are using a distribution's packages and want to alter the log rotation, check `/etc/logrotate.d/` for a certbot rotation script.

<a id="command-line"></a>

## Certbot 命令行选项

Certbot supports a lot of command line options. Here's the full list, from `certbot --help all`:

.. literalinclude:: cli-help.txt

## 得到帮助

如果您有问题，我们建议您在 Let's Encrypt[社区论坛](https://community.letsencrypt.org)上发布。.

如果您在软件中发现 bug，请在我们的[问题跟踪器](https://github.com/certbot/certbot/issues)中报告。
记得给我们尽可能多的信息:

- 复制并粘贴所使用的命令行和输出(注意后者可能包含一些个人身份信息，包括您的电子邮件和域名)
- 从`/var/log/letsencrypt`中复制并粘贴日志(尽管注意它们也可能包含个人身份信息)
- 复制并粘贴`certbot --version`输出
- 您的操作系统，包括特定版本
- 指定您选择的安装方法
