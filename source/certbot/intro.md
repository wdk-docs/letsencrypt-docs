# 简介

:link: <https://eff-certbot.readthedocs.io/en/stable/index.html>

!!! Note

    要快速入门，请使用[交互式安装指南](https://certbot.eff.org/)。

Certbot 是 EFF 为整个互联网加密所做努力的一部分。
网络上的安全通信依赖于 HTTPS，它需要使用数字证书，让浏览器验证网络服务器的身份(例如，这真的是 google.com 吗?)
Web 服务器从被称为证书颁发机构(ca)的受信任第三方获取证书。
Certbot 是一个易于使用的客户端，它可以从 Let’s encrypt(一个由 EFF、Mozilla 和其他机构推出的开放证书颁发机构)获取证书，并将其部署到 web 服务器上。

任何经历过建立安全网站的人都知道获得和维护证书有多麻烦。
Certbot 和 Let 's Encrypt 可以自动消除这种痛苦，让您通过简单的命令打开和管理 HTTPS。
使用 Certbot 和 Let 's Encrypt 是免费的，所以不需要安排付款。

你如何使用 Certbot 取决于你的网络服务器的配置。最好的方法是使用我们的互动指南。
它根据您的配置设置生成指令。
在大多数情况下，您需要 root 或管理员访问您的 web 服务器才能运行 Certbot。

Certbot 可以直接在你的网络服务器上运行，而不是在你的个人电脑上。
如果您正在使用托管服务，并且不能直接访问 web 服务器，则可能无法使用 Certbot。
请向托管提供商查询有关上传证书或使用 Let 's Encrypt 颁发的证书的文档。

Certbot 是 Let 's Encrypt CA(或任何其他使用 ACME 协议的 CA)的功能齐全、可扩展的客户端，可以自动化获取证书和配置 web 服务器以使用它们的任务。
该客户端运行在基于 unix 的操作系统上。

要查看 Certbot 在不同版本之间所做的更改，请参阅我们的更新日志。

## 贡献

如果你想为这个项目做贡献，请阅读开发者指南。

本项目受 EFF 公共项目行为准则的约束。

## 如何运行客户端

安装和运行 Certbot 最简单的方法是访问 <certbot.eff.org>，在那里您可以找到许多 web 服务器和操作系统组合的正确说明。
有关更多信息，请参见 [Get Certbot](https://certbot.eff.org/docs/install.html)。

## 更深入地了解客户

要详细理解客户端在做什么，了解它使用插件的方式很重要。
请参见用户指南中对[插件的说明](https://certbot.eff.org/docs/using.html#plugins)。

### 链接

- 文档: <https://certbot.eff.org/docs>
- 软件项目: <https://github.com/certbot/certbot>
- 开发人员注意事项: <https://certbot.eff.org/docs/contributing.html>
- 主要的网站: <https://certbot.eff.org>
- Let’s Encrypt 网站: <https://letsencrypt.org>
- 社区: <https://community.letsencrypt.org>
- ACME spec: [RFC 8555](https://tools.ietf.org/html/rfc8555)
- github 中的 ACME 工作区(存档): <https://github.com/ietf-wg-acme/acme>

### 系统需求

查看 <https://certbot.eff.org/docs/install.html#system-requirements>.
