# 获取 Certbot

<a id="system_requirements"></a>

## 系统需求

- Linux, macOS, BSD and Windows
- Linux/BSD 推荐 root 权限，Windows 需要 Administrator 权限
- 80 端口开放

!!! Note

    Certbot在root权限下运行时最有用，因为它能够自动为Apache和nginx配置TLS/SSL。

_Certbot 可以直接在网络服务器上运行_, 通常由系统管理员负责。在大多数情况下，在个人计算机上运行 Certbot 并不是一个有用的选择。下面的说明涉及在服务器上安装和运行 Certbot。

## 安装

除非您有非常特殊的要求，否则我们建议您使用 <https://certbot.eff.org/instructions> 上的系统安装说明。

<a id="snap-install"></a>

## Snap (推荐)

我们的指令在所有使用 Snap 的系统中都是相同的。
您可以在 <https://certbot.eff.org/instructions> 上找到通过 Snap 安装 Certbot 的说明，选择您的服务器软件，然后在"System"下拉菜单中选择“snapd”。

大多数现代 Linux 发行版(基本上任何使用 systemd 的发行版)都可以安装打包成 snap 的 Certbot。
snap 可用于 x86_64、ARMv7 和 ARMv8 架构。
Certbot snap 提供了一种简单的方法来确保您拥有最新版本的 Certbot，并预先配置了自动证书更新等功能。

如果你不能使用 snaps，你可以使用另一种方法来安装`certbot`。

<a id="docker-user"></a>

## 选择 1: Docker

Docker 是一种非常简单快速的获取证书的方法。
但是，这种操作模式无法安装证书或配置您的 web 服务器，因为我们的安装程序插件无法从 Docker 容器内部到达您的 web 服务器。

大多数用户应该使用[certbot.eff.org]上的说明。
只有当你确定自己在做什么并且有充分的理由这样做时，你才应该使用 Docker。

您一定要阅读`where-certs`部分，以便了解如何手动管理证书。['我们的密码套件页面'](ciphersuite .html)
提供有关推荐的密码套件的一些信息。
如果这些对您来说都不太有意义，那么您一定要使用[certbot.eff.org]为您的系统推荐的安装方法，该方法允许您使用涵盖这两个困难主题的安装程序插件。

如果您仍然不相信并决定使用此方法，则从您正在请求证书的域的服务器解析到[`install Docker`]，然后发出如下所示的命令。
如果你使用的是带有`Standalone`插件的 Certbot，你需要在`certbot/certbot`之前的命令行中包含`-p 80:80` 或 `-p 443:443`之类的内容，从而使它所使用的端口从容器外部可访问。

```shell

sudo docker run -it --rm --name certbot \
 -v "/etc/letsencrypt:/etc/letsencrypt" \
 -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
 certbot/certbot certonly
```

使用`certonly`命令运行 Certbot 将获得一个证书，并将其放在系统上的`/etc/letsencrypt/live`目录下。
因为 Certonly 不能从 Docker 内部安装证书，所以您必须按照 web 服务器提供商推荐的步骤手动安装证书。

在<https://hub.docker.com/u/certbot>上也有 Certbot 的每个 DNS 插件的 Docker 映像，可以自动为流行的提供商进行 DNS 域验证。
要使用一个，只需将上面命令中的`certbot/certbot`替换为您想要使用的映像的名称。
例如，要在 Amazon Route 53 上使用 Certbot 的插件，你可以使用`certbot/dns-route53`。
您可能还需要向 Certbot 添加标志和/或挂载其他目录，以提供对[DNS 插件文档](dns_plugins)中指定的 DNS API 凭据的访问。

有关`/etc/letsencrypt`目录布局的更多信息，请参阅[`where-certs`]。

[docker]: https://docker.com
[`install docker`]: https://docs.docker.com/engine/installation/
[certbot.eff.org]: https://certbot.eff.org/instructions

<a id="pip"></a>

## 选择 2: Pip

Installing Certbot through pip is only supported on a best effort basis and
when using a virtual environment. Instructions for installing Certbot through
pip can be found at https://certbot.eff.org/instructions by selecting your
server software and then choosing "pip" in the "System" dropdown menu.

<a id="third-party"></a>

## 选择 3: 第三方分发

Third party distributions exist for other specific needs. They often are maintained
by these parties outside of Certbot and tend to rapidly fall out of date on LTS-style distributions.

<a id="certbot"></a>

## Certbot-Auto [作废]

.. toctree::
:hidden:

We used to have a shell script named `certbot-auto` to help people install
Certbot on UNIX operating systems, however, this script is no longer supported.

Please remove `certbot-auto`. To do so, you need to do three things:

1. If you added a cron job or systemd timer to automatically run certbot-auto to renew your certificates, you should delete it. If you did this by following our instructions, you can delete the entry added to `/etc/crontab` by running a command like `sudo sed -i '/certbot-auto/d' /etc/crontab`.
2. Delete the certbot-auto script. If you placed it in ` /usr/local/bin`` like we recommended, you can delete it by running  `sudo rm /usr/local/bin/certbot-auto`.
3. Delete the Certbot installation created by certbot-auto by running `sudo rm -rf /opt/eff.org`.
