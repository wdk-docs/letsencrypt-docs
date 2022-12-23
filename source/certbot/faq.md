# 常见问题

!!! Question "我的浏览器信任 Let 's Encrypt 提供的证书吗?"

    对于大多数浏览器和操作系统来说，是的。有关更多详细信息，请参阅兼容性列表。

!!! Question "Certbot 会为网站颁发 SSL/TLS 以外的证书吗?"

    Certbot将获取Let 's Encrypt证书，这些证书将是标准的域验证证书，因此您可以将它们用于任何使用域名的服务器，如web服务器。
    您还可以将这些证书用于IMAPS等其他TLS应用程序。

!!! Question "我可以使用 Certbot 的证书进行代码签名或电子邮件加密吗?"

    不。电子邮件加密和代码签名需要与Let's Encrypt CA颁发的证书类型不同的证书。

!!! Question "Certbot 会在 Let’s Encrypt 的服务器上生成或存储我的证书的私钥吗?"

    不。从来没有。

    私钥总是在您自己的服务器上生成和管理，而不是由Let’s Encrypt证书颁发机构生成和管理。

!!! Question "Certbot 会颁发扩展验证(EV)证书吗?"

    Certbot and Let’s Encrypt have no plans to issue EV certificates at this time.

!!! Question "我可以获得多个域名的证书(SAN 证书)吗?"

    Yes, the same certificate can apply to several different names using the Subject Alternative Name (SAN) mechanism. Certbot automatically requests certificates for multiple names when requested to do so. The resulting certificates will be accepted by browsers for any of the domain names listed in them.

!!! Question "Let's Encrypt 颁发通配符证书吗?"

    Yes! Let's Encrypt has begun issuing wildcard certificates in March 2018. Certbot has added support for wildcard certificates as of version 0.22.0. Obtaining a wildcard certificate requires using the DNS authentication method, either via --manual or via a Certbot DNS plugin appropriate to your DNS provider.

    Note that depending how you install Certbot, appropriate plugins to automate the process may not yet be available on your system. Information about the DNS plugins is available in the Certbot documentation.

    Certificates obtained with --manual cannot be renewed automatically with certbot renew (unless you've provided a custom authorization script). However, certificates obtained with a Certbot DNS plugin can be renewed automatically. In order to obtain wildcard certificates that can be renewed without human intervention, you'll need to use a Certbot DNS plugin that's compatible with an API supported by your DNS provider, or a script that can make appropriate DNS record changes upon demand. Even if your regular DNS provider doesn't support a compatible update mechanism, you can use a CNAME delegation for the \_acme-challenge record in your DNS zone to a different provider that does. You can also point \_acme-challenge to an acme-dns instance.

    Note that depending how you install Certbot, appropriate plugins to automate the process may not yet be available on your system.

    Please see Certbot documentation for more information about your situation.

!!! Question "Certbot 支持我的操作系统吗?"

    We currently have Certbot support for major Linux and BSD variant operating systems. There are a large number of other client implementations available too.

!!! Question "Certbot 是否支持自动配置我的 web 服务器?"

    This website provides information about the level of support for various web servers and operating systems, which varies and is increasing over time. On supported systems, the automated configuration makes it fast and easy to obtain, install, and automatically renew certificates.

    If automated configuration is not supported for your web server, you can still get a certificate using Certbot and configure your server software manually. In this case, the certificate will not be renewed automatically.

    Note that automated configuration is not required. It can be disabled if you prefer to configure your server software yourself.

!!! Question "Certbot 需要 root/管理员权限吗?"

    Whether root is required to run Certbot or not depends on how you intend to use it.

    If you're asking this question because you have a hosting provider that doesn't grant you root access, you'll need to ensure first of all that you have a way to install a certificate if you get one. If the answer is "no", ask your hosting provider to support Let's Encrypt (many already do). If the answer is "yes", or you're asking the question for security reasons, read on...

    The webroot and manual plugins work well without root privileges. However, you need to provide writable paths for Certbot's working directories either by ensuring that /etc/letsencrypt/, /var/log/letsencrypt/, /var/lib/letsencrypt/ are writable, or by picking different directories with the --config-dir, --logs-dir, and --work-dir flags.

    The standalone plugin requires root to bind port 80 or 443, although on Linux you could also grant CAP_NET_BIND_SERVICE to the relevant user.

    Certbot's Apache and Nginx plugins normally require root both for making temporary and persistent changes to webserver configurations, and to perform graceful reload events for those servers.

    The certbot-auto script works on the assumption that root privileges will be used, both in order to install OS dependencies where required and because it needs to support all of the plugins mentioned above. The packaged versions of Certbot are more flexible, and some of the teams building these packages are working toward having Cerbot run with group rather than root privileges where possible.

!!! Question "我可以在 Certbot 上使用现有的私钥或证书签名请求(CSR)吗?"

    Yes. You can obtain a certificate for an existing CSR, which means you may generate your own CSR using your own private key. However, Certbot will not accept a private key as input and generate a CSR for you.

!!! Question "目前的利率限制是什么?"

    https://letsencrypt.org/docs/rate-limits/

!!! Question "我可以在不关闭我的 web 服务器的情况下颁发证书吗?"

    Yes, Certbot has different plugins to perform domain validation and none of them require any downtime except for the "standalone" plugin.

!!! Question "“Let's Encrypt”服务器将使用哪些 IP 地址来验证我的 web 服务器?"

    The Let's Encrypt CA doesn't publish a list of IP addresses it uses to validate, because they may change at any time. In the future, it may validate from multiple IP addresses at once.

!!! Question "如果我的 web 服务器没有监听端口 80，我可以颁发证书吗?"

    Yes, using the DNS-01 or TLS-ALPN-01 challenge. However, Certbot does not include support for TLS-ALPN-01 yet. If you're using any Certbot with any method other than DNS authentication, your web server must listen on port 80, or at least be capable of doing so temporarily during certificate validation.

    If you have an ISP or firewall that blocks port 80 and you can't get it unblocked, you'll need to use DNS authentication or a different Let's Encrypt client.

!!! Question "我可以使用什么工具来调试站点的 HTTPS 配置?"

    There are four scanning tools that are commonly suggested on the Let’s Encrypt community forum:

    https://letsdebug.net/ (by Alex Zorin)
    https://check-your-website.server-daten.de/ (by Jürgen Auer)
    https://whynopadlock.com/ (by LexiConn)
    https://www.ssllabs.com/ssltest/ (by Qualys)
    They all have their strengths. Let's Debug would be used only by people who don't have HTTPS working yet, while SSL Labs would be used only by people who (at least sort of) do.

    Let’s Debug: Let's Debug is most helpful if you have a failed challenge and want a straightforward explanation of why the challenge is failing.

    Check-Your-Website: Jürgen's scanner is most helpful if you have a confusing DNS or HTTP configuration error where some pages or some browsers work properly and others don't, or if your HTTP site works in a browser and yet you get failed challenges that you don't understand

    Why No Padlock: Why No Padlock is most helpful if you already have a certificate but all or some users don't see a valid HTTPS connection (and it gives very specific information about what's causing mixed content warnings)

    SSL Labs: SSL Labs is most helpful for cryptographic issues on an already set up HTTPS site, such as a case where some browsers work properly and others give a ciphersuite-related error, or if you want to convince nerds and/or regulatory bodies that you're following security best practices

!!! Question "Certbot 的隐私政策是什么?"

    The Certbot privacy policy can be found here.

!!! Question "Certbot 和这个网站的授权是什么?"

    The Certbot software and documentation are licensed under the Apache 2.0 license as described here. Otherwise, this website is generally licensed under EFF's CC-BY license, except this FAQ page, which is a derivative of the Let’s Encrypt FAQ (which was licensed under Let’s Encrypt’s CC-BY-NC).
