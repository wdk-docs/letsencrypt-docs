# 什么是证书?

公钥或数字证书(以前称为 SSL 证书)使用公钥和私钥在客户端程序(web 浏览器、电子邮件客户端等)和服务器之间通过加密的 SSL(安全套接字层)或 TLS(传输层安全)连接实现安全通信。
证书既用于加密通信的初始阶段(安全密钥交换)，也用于标识服务器。
证书包括有关密钥的信息、有关服务器身份的信息以及证书颁发者的数字签名。
如果发起通信的软件信任颁发者，并且签名有效，则可以使用该密钥与由证书标识的服务器安全地通信。
使用证书是防止“中间人”攻击的好方法，在这种攻击中，位于您和您认为正在与之对话的服务器之间的某人能够插入他们自己的(有害的)内容。

您可以使用 Certbot 轻松地从 Let’s Encrypt 获取和配置免费证书，Let’s Encrypt 是 EFF、Mozilla 和许多其他赞助商的联合项目。

## 证书和血统

Certbot 引入了沿袭的概念，它是证书的所有版本的集合，加上从更新到更新期间为该证书维护的 Certbot 配置信息。
当您更新证书时，Certbot 将保持相同的配置，除非您显式地更改它，例如添加或删除域。
如果您添加域，您可以将它们添加到现有沿袭或创建一个新的沿袭。

另请参阅: 重新创建和更新已有证书
