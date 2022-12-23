# 我的 HTTP 网站在 macOS 上运行 Nginx

1.  SSH 进入服务器

    以具有 sudo 权限的用户 SSH 到运行 HTTP 网站的服务器。

2.  安装 Homebrew

    您需要安装 Homebrew。

    在 Homebrew 网站上按照以下说明安装 Homebrew。

    [安装 Homebrew](https://brew.sh/)

3.  安装 Certbot

    在机器的命令行上运行此命令来安装 Certbot。

    ```sh
    brew install certbot
    ```

4.  选择你想如何运行 Certbot

    **你可以暂时停止你的网站吗?**

    ???+ Success "是的，我的 web 服务器目前没有在这台机器上运行。"

        停止您的web服务器，然后运行此命令获取证书。Certbot会暂时启动你机器上的网络服务器。

        ```sh
        sudo certbot certonly --standalone
        ```

    ???+ Success "不，我需要保持我的网络服务器正常运行。"

        如果您的web服务器已经在使用端口80，并且不想在Certbot运行时停止它，请运行此命令并遵循终端中的说明。

        ```sh
        sudo certbot certonly --webroot
        ```

        !!! note

            要使用webroot插件，您的服务器必须配置为提供隐藏目录中的文件。
            如果您的web服务器配置对`/.well-known`进行了特殊处理，您可能需要修改配置以确保`/.well-known/acme-challenge`内的文件由web服务器提供。

5.  安装证书

    您需要在 web 服务器的配置文件中安装新证书。

6.  设置自动续订

    我们建议运行下面的代码行，它将向默认的 crontab 添加一个 cron 作业。

    ```sh
    echo "0 0,12 * * * root $(command -v python3) -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo $(command -v certbot) renew -q" | sudo tee -a /etc/crontab > /dev/null
    ```

7.  确认 Certbot 工作正常

    要确认您的站点已正确设置，请在浏览器中访问 <https://yourwebsite.com/> ，并在 URL 栏中寻找锁图标。
