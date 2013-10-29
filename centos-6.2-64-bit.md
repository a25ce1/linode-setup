1. 设置反向域名解析、局域网 IP，然后重启服务器
----------------------------------------------

2. 设置主机名
-------------

    echo 'HOSTNAME=plato' >> /etc/sysconfig/network
    hostname 'plato'

3. 编辑 /etc/hosts
------------------

    127.0.0.1       localhost.localdomain   localhost
    12.34.56.78     plato.example.com       plato

4. 设置时区
-----------

    ln -sf /usr/share/zoneinfo/UTC /etc/localtime

5. 安装 EPEL 和 Remi，并安装软件更新
------------------------------------

    rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
    wget https://raw.github.com/a25ce1/aliyun-setup/master/etc/remi.repo -O /etc/yum.repos.d/remi.repo
    yum update -y

