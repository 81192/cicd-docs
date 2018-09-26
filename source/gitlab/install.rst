安装
+++++++

RHEL/CentOS
"""""""""""""""

1. 安装并配置必要的依赖项

    在 CentOS7/RHEL 系统中，下面的命令将会在系统防火墙中开放 HTTP/HTTPS 和 SSH 端口

    .. code-block:: bash

        sudo yum install -y curl policycoreutils-python openssh-server
        sudo systemctl enable sshd
        sudo systemctl start sshd
        sudo firewall-cmd --permanent --add-service=http
        sudo firewall-cmd --permanent --add-service=https
        sudo systemctl reload firewalld

    接下来，安装Postfix以发送通知电子邮件。如果要使用其他解决方案发送电子邮件，请跳过此步骤并在安装GitLab后配置外部SMTP服务器。

    .. code-block:: bash

        sudo yum install postfix
        sudo systemctl enable postfix
        sudo systemctl start postfix

    在Postfix安装期间，可能会出现配置界面。选择“Internet Site”并按Enter键。使用服务器的外部DNS作为“邮件名称”，然后按Enter键。如果出现其他界面，请继续按Enter键接受默认值。

2. 添加 GitLab 软件包存储库并安装软件包

    .. code-block:: bash

        curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

