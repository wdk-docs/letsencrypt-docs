# certbot 术语

!!! Quote "Admin Access"

    如果您是能够控制服务器并执行诸如安装和删除软件或整体更改系统等管理任务的人，那么您就具有对服务器的管理访问权。
    这通常也被称为`根访问`。如果您可以成功地在服务器上运行`sudo`，那么您就拥有管理员访问权限。
    如果您使用VPS或专用服务器，或者其他人已经为您设置了服务器，则可能会出现这种情况。
    如果您使用共享服务器，或者您只能通过FTP、SFTP或控制面板接口与服务器交互，则可能不会出现这种情况。

!!! Quote "Automatic Renewal"

    In the context of Certbot, this refers to a feature designed to automatically obtain new versions of a certificate before it expires. This feature is important as helps you avoid gaps between having valid certificates that can be trusted by the users of your server.

!!! Quote "Automatic Updates"

    Automatic updates are when a piece of software downloads a new version of itself automatically, usually as soon as practical after it becomes available. This process is designed to help the software have the latest bug fixes and features that have been released by the developer.

!!! Quote "Backports"

    Backports is a feature in some operating systems that provides an option for getting newer versions of software than those that your operating system package manager would otherwise provide you. The installation instructions for Certbot in some environments may ask you to follow instructions for enabling the backports feature on your operating system.

!!! Quote "Browser"

    A browser is the program you use to view websites on the Internet. Firefox, Safari, Internet Explorer, and Chrome are all web browsers. Mobile devices have a web browser app for the same purpose.

!!! Quote "Certificate"

    A certificate is a file that mathematically shows browsers or other software that they’ve made an encrypted connection to the site they attempted to connect to.

!!! Quote "Certificate Authority"

    A certificate authority is an organization that gives out certificates to website operators.

!!! Quote "Cloud Hosting"

    Cloud hosting can refer to any situation in which your web site is hosted using someone else's infrastructure, typically on servers belonging to a web hosting company. This could be contrasted with a web site that's hosted on your own personal server, such as a physical machine running in your home.

!!! Quote "Command Line"

    A command line is a way of interacting with a computer by typing text-based commands to it and receiving text-based replies. Certbot is run from a command-line interface, usually on a Unix-like server. In order to use Certbot for most purposes, you’ll need to be able to install and run it on the command line of your web server, which is usually accessed over SSH.

!!! Quote "Control Panel"

    A control panel is a type of software that provides an interface for configuring websites, such as cPanel.

!!! Quote "Cron Job"

    A cron job is a task to be run at a scheduled time by the program cron.

!!! Quote "Crontab"

    The `cron table` is a specific file that contains tasks to be run at a scheduled time by the program cron.

!!! Quote "DNS"

    DNS is the Domain Name System which creates a worldwide directory of domain names, like example.com, certbot.eff.org, community.letsencrypt.org, or millions of others. These domain names can be looked up by Internet users’ software anywhere in the world to learn IP addresses and other technical data that’s used to make connections to servers. DNS is an important part of Internet infrastructure and is operated collectively by hundreds of organizations, including DNS registrars (which let you pay to register your own domain name for your own site) and various DNS providers.

!!! Quote "DNS Credentials"

    DNS credentials are a password or other kind of secret (such as an API key) that your DNS provider lets you use to change the contents of your DNS records. They are usually issued by your domain registrar (or by another DNS provider, if your DNS provider isn’t the same as your registrar). DNS credentials are a sensitive kind of secret because they can be used to take over your site completely. You should never share these credentials publicly or with an unauthorized person. It can be OK to provide a copy of them to Certbot to let it perform DNS validation automatically, since it runs locally on your machine.

!!! Quote "DNS Provider"

    A Domain Name System (DNS) provider is an organization that runs DNS servers (also called nameservers) to host DNS records for domain names. Your DNS provider could be the same as, or different from, your DNS registrar (whom you pay to register your domain name), or your hosting provider (whom you pay to host your web site). You can also change your DNS provider if your current provider doesn’t offer the features that you want. Your DNS provider must follow various Internet technical standards in order for Let’s Encrypt to confirm that it’s you who controls your domain name when you request a certificate. You need to interact with your DNS provider in order to change the DNS records that refer to your domain name. Some DNS providers can help you create Let’s Encrypt certificates (including wildcard certificates) easily using Let’s Encrypt’s DNS validation method, including by providing a software interface that Certbot can use for updating DNS records automatically.

!!! Quote "DNS Record"

    A DNS record is an entry in a nameserver that tells machines on the Internet some information about the domain. For example, an `A record` describes which IPv4 address a particular domain name should point to.

!!! Quote "DNS Validation"

    DNS validation is a method to prove your control over your domain name by asking you to create specific DNS TXT records within the domain. This requires you to have appropriate credentials to change your DNS records. DNS validation is one of the options provided by Let’s Encrypt to prove your control over your domain name. If you have a DNS provider that’s supported by Certbot, Certbot can automate this process. Otherwise, you would have to perform it yourself every time you renew your certificate. Most Certbot users don’t need to perform DNS validation, but it’s required by Let’s Encrypt if you want a wildcard certificate, and it can also be a useful option if your web server can’t receive connections from the Internet (for example, because it’s on a private network behind a firewall).

!!! Quote "Docker"

    Docker is a system that allows website operators to run pre-configured systems called `containers` inside of their server. These containers come with everything needed to perform a task. Certbot’s Docker containers ship with Certbot already set up inside of them.

!!! Quote "Domain Name"

    A domain name is a human-readable identifier for a website that someone can type into the URL bar of their browser to go to that website. The domain name doesn’t include the parts of the URL after the first single slash mark (/).

!!! Quote "Domain Registrar"

    A domain registrar is an organization that website operators purchase domains from.

!!! Quote "Firewall"

    A firewall monitors and controls network traffic on a machine.

!!! Quote "FTP"

    The File Transfer Protocol (FTP) allows users to move files from one machine to another over the network.

!!! Quote "Hosting Provider"

    A hosting provider is a company or other organization that helps you get your website online. Hosting providers are usually paid by a customer and offer a variety of different plans and services, including shared hosting, virtual private servers, and dedicated servers. Their servers are usually located in data centers, and the sites or servers they host are usually administered remotely over the Internet.

!!! Quote "HTTP"

    HTTP (Hypertext Transfer Protocol) is the traditional, but insecure, method for web browsers to request the content of web pages and other online resources from web servers. It is an Internet standard and normally used with TCP port 80. Almost all websites in the world support HTTP, but websites that have been configured with Certbot or some other method of setting up HTTPS may automatically redirect users from the HTTP version of the site to the HTTPS version.

!!! Quote "HTTPS"

    HTTPS (Hypertext Transfer Protocol Secure) is the update to HTTP that uses the SSL/TLS protocol to provide security for connections between web browsers and web servers. Using HTTPS normally requires a certificate from a certificate authority, such as Let’s Encrypt, and will also require installing that certificate onto a web server. Certbot can help perform both of these steps automatically in many cases. HTTPS is an Internet standard and is normally used with TCP port 443.

!!! Quote "Internet Service Provider"

    An Internet Service Provider (ISP) is an organization that people can obtain access to the Internet from.

!!! Quote "IP Address"

    An IP address is a group of numbers, like 198.51.100.55, that identify a computer on the Internet and allow other computers to talk to it.

!!! Quote "Let’s Encrypt"

    Let’s Encrypt is a certificate authority that issues digital certificates free of charge to let people get encrypted HTTPS connections to web sites on the Internet, which are substantially more secure than unencrypted connections. Certbot is a software tool made by the Electronic Frontier Foundation. Certbot is the most popular way for people who run their own web servers to get a Let’s Encrypt certificate, set up HTTPS on the server, and renew the certificate automatically in the future. There are also many other tools and options for people in different situations to get Let’s Encrypt certificates.

!!! Quote "Operating System"

    The operating system is the core software running on a computer. The most common operating systems are Windows, macOS, and Linux. Linux is subdivided into `distributions` like Ubuntu, Debian, RedHat, and many more.

!!! Quote "Port 443"

    Port 443 is the port officially assigned for use with HTTPS. Different Internet services are distinguished by using different TCP port numbers. Unencrypted HTTP normally uses TCP port 80, while encrypted HTTPS normally uses TCP port 443.

!!! Quote "Port 80"

    Different Internet services are distinguished by using different TCP port numbers. Unencrypted HTTP normally uses TCP port 80, while encrypted HTTPS normally uses TCP port 443. To use certbot --webroot, certbot --apache, or certbot --nginx, you should have an existing HTTP website that’s already online hosted on the server where you’re going to use Certbot. This site should be available to the rest of the Internet on port 80. To use certbot --standalone, you don’t need an existing site, but you have to make sure connections to port 80 on your server are not blocked by a firewall, including a firewall that may be run by your Internet service provider or web hosting provider. Please check with your ISP or hosting provider if you’re not sure. (Using DNS validation does not require Let’s Encrypt to make any inbound connection to your server, so with this method in particular it’s not necessary to have an existing HTTP website or the ability to receive connections on port 80.)

!!! Quote "Server"

    A server is a computer on the Internet that provides a service, like a web site or an email service. Most web site owners pay a hosting provider for the use of a server located in a data center and administered over the Internet. This might be a physical dedicated server, a virtual private server (VPS), or a shared server. Other servers provide other parts of the Internet infrastructure, such as DNS servers.

!!! Quote "Server - Dedicated Server"

    A dedicated server is a server that only hosts the contents or services for a single website administrator.

!!! Quote "Server Plugin"

    In the context of Certbot, this refers to software that works with Certbot to configure a specific piece of server software. For instance, Certbot has an Apache plugin and a nginx plugin which can be used to obtain and configure certificates with those servers.

!!! Quote "Server - Shared Server"

    A shared server is a server that hosts content or services run by different people on the same machine. A popular version of this is shared hosting, where websites run by different people are provided by the same server that is shared between them.

!!! Quote "Server - Virtual Private Server (VPS)"

    A virtual private server is a complete server environment within which the customer can control the entire operating system and software environment. That allows the customer to be the system administrator for this server environment. This is the second-most common kind of web hosting environment (following shared hosting), and is offered by major providers like Amazon AWS, Azure, DigitalOcean, and Linode, among others. Most successful Certbot users are running Certbot in a VPS environment.

!!! Quote "Server - Web Server"

    `Web server` can refer to either the server (computer), application (software), or the combination of the two that’s used to provide your web site’s content to the rest of the Internet.

!!! Quote "SFTP"

    SFTP is a more-secure version of FTP.

!!! Quote "Shared Hosting"

    Shared hosting is a kind of web hosting in which several customers’ web sites are hosted by the same server, which is mainly administered by the web hosting provider. Usually customers administer their own sites on a shared server using a web-based control panel interface. This is less expensive than VPS hosting and often calls for less technical knowledge on the customer’s part; overall, it’s the most popular type of web hosting environment.

    Certbot is less suitable for use in most shared hosting environments because it’s usually easier and more reliable to ask the hosting provider to set up HTTPS. (Some shared hosting users use Certbot, most often because their hosting providers are uncooperative or don’t have another way to enable HTTPS support for customer sites.)

!!! Quote "Software"

    Software is a set of instructions that teach a computer how to perform a particular task. Certbot is one software application that can be useful for web site administrators who want to set up HTTPS on their web sites. Certbot documentation will also expect you to know the names and versions of some other software that you use on your web server. For instance, the way to install Certbot is different on different operating systems, so you'll be asked the operating system software that your web server uses.

!!! Quote "Software repository or `repo`"

    A repo is a place where software is stored to be accessed by others. Examples of this include a GitHub repository that contains the files for a software project, or the repositories offered by popular operating systems such as Debian, Fedora, and Ubuntu, which contain many different pieces of software for their users to download and install.

!!! Quote "SSH"

    SSH (which stands for `secure shell`) is a technology for connecting to a remote server and accessing a command line on that server, often in order to administer it. The administrator of a server can grant SSH access to others, and can also use SSH access directly in order to administer the server remotely. SSH is usually used to access servers running Unix-like operating systems, but your own computer doesn’t have to be running Unix in order to use SSH. You normally use SSH from your computer’s command line in a terminal by typing a command such as ssh username@example.com, especially if your own computer runs Linux or macOS. After logging in, you’ll have access to the server’s command line. If you use Windows on your computer, you might also use a dedicated SSH application such as PuTTY. Most Certbot users run Certbot from a command prompt on a remote server over SSH.

!!! Quote "SSL/TLS"

    SSL, aka TLS, is a protocol used to encrypt data sent between two computers. When you browse the web with encryption using HTTPS (instead of HTTP), your web browser is using SSL/TLS to talk to web sites under the hood. Sometimes SSL and TLS are used to distinguish different protocol versions, but for most common purposes they are synonyms.

!!! Quote "sudo"

    Sudo is the most common command on Unix-like operating systems to run a specific command as root (the system administrator). If you’re logged in to your server as a user other than root, you’ll likely need to put sudo before your Certbot commands so that they run as root (for example, sudo certbot instead of just certbot), especially if you’re using Certbot’s integration with a web server like Apache or Nginx. (The certbot-auto script automatically runs sudo if it’s necessary and you didn’t specify it.)

!!! Quote "TCP"

    TCP is a protocol that allows two computers to send data to each other. More complex protocols like HTTP, HTTPS, SSH, and FTP, are built using TCP as a building block.

!!! Quote "Updates"

    It is important to keep software up-to-date so that the latest security fixes and features can be applied.

!!! Quote "Website That’s Already Online"

    Certbot is usually meant to be used to switch an existing HTTP site to work in HTTPS (and, afterward, to continue renewing the site’s HTTPS certificates whenever necessary). Some Certbot documentation assumes or recommends that you have a working web site that can already be accessed using HTTP on port 80. That means, for example, that if you use a web browser to go to your domain using http://, your web server answers and some kind of content comes up (even if it’s just a default welcome page rather than the final version of your site). Some methods of using Certbot have this as a prerequisite, so you’ll have a smoother experience if you already have a site set up with HTTP. (If your site can’t be accessed this way as a matter of policy, you’ll probably need to use DNS validation in order to get a certificate with Certbot.)

!!! Quote "Wildcard Certificate"

    通配符证书是包含一个或多个以`*.`开头的名称的证书。
    浏览器将接受任何替代星号(*)的标签。
    例如，`*.example.com`的证书将对`www.example.com`、`mail.example.com`、`hello.example.com`和`.goodbye.example.com`有效。

    但是，只包含名称`*.example.com`的通配符证书对`example.com`无效:替换的标签不能为空。
    如果您希望证书对`example.com`有效，还需要包括`example.com`(即不带 `*.` 部分)。

    此外，星号只能由一个标签代替，不能由多个标签代替。
    例如，名称`hello.goodbye.example.com`将不会被仅包含名`*.example.com`的证书覆盖。
    但是，它将被`*.goodbye.example.com`覆盖。
    注意，通配符名称不能包含多个星号。
    例如，`*.*.example.com`无效。
