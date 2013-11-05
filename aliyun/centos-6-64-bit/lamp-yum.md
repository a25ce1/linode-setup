# 阿里云 CentOS 6.3 64 位安全加固版 LAMP 快速安装教程

成功安装后的网站根目录和一些配置文件的路径

* Apache 网站根目录 `/var/www/html`
* 配置文件
    * Apache `/etc/httpd/conf/httpd.conf` 和 `/etc/httpd/conf.d/*.conf`
    * MySQL `/etc/my.cnf`
    * PHP `/etc/php.ini`

## 挂载数据盘

### 创建分区

    fdisk /dev/xvdb
    n
    p
    1
    （回车）
    （回车）
    w

### 格式化分区

    mkfs.ext4 /dev/xvdb1

### 挂载分区到 `/data`

    mkdir /data
    echo '/dev/xvdb1              /data                   ext4    defaults        0 0' >> /etc/fstab
    mount -a

## 安装 EPEL 和 Remi，并安装软件更新

    rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    
    rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
    wget https://raw.github.com/a25ce1/server-setup/master/etc/remi.repo -O /etc/yum.repos.d/remi.repo
    
    yum update -y

## 创建防火墙

    vi /etc/iptables.firewall.rules

复制、粘贴如下内容：

    *filter
    
    #  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
    -A INPUT -i lo -j ACCEPT
    -A INPUT -d 127.0.0.0/8 -j REJECT
    
    #  Accept all established inbound connections
    -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    
    #  Allow all outbound traffic - you can modify this to only allow certain traffic
    -A OUTPUT -j ACCEPT
    
    #  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
    -A INPUT -p tcp --dport 80 -j ACCEPT
    -A INPUT -p tcp --dport 443 -j ACCEPT
    
    #  Allow SSH connections
    #
    #  The -dport number should be the same port number you set in sshd_config
    #
    -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
    
    #  Allow ping
    -A INPUT -p icmp -j ACCEPT
    
    #  Log iptables denied calls
    -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7
    
    #  Drop all other inbound - default deny unless explicitly allowed policy
    -A INPUT -j DROP
    -A FORWARD -j DROP
    
    COMMIT

### 应用防火墙规则

    iptables-restore < /etc/iptables.firewall.rules

### 保存当前防火墙规则

    /sbin/service iptables save

### 设置防火墙开机自动启动

    service iptables on

## 安装和配置 Fail2Ban

    yum install -y fail2ban
    
    wget https://raw.github.com/a25ce1/server-setup/master/etc/jail.local -O /etc/fail2ban/jail.local
    
    chkconfig fail2ban on

## 安装 Apache 服务器

    yum install -y httpd
    service httpd start
    chkconfig httpd on

### 配置

参考：https://github.com/h5bp/server-configs-apache/tree/master/doc

`Options -Indexes`

`AllowOverride All`

    rm -f /etc/httpd/conf.d/welcome.conf
    wget https://raw.github.com/a25ce1/server-setup/master/etc/_default.conf -O /etc/httpd/conf.d/_default.conf
    wget https://raw.github.com/a25ce1/server-setup/master/etc/_sample.conf -O /etc/httpd/conf.d/_sample.conf
    mkdir /var/www/html/_default
    wget https://raw.github.com/a25ce1/server-setup/master/etc/index.html -O /var/www/html/_default/index.html
    wget https://raw.github.com/a25ce1/server-setup/master/etc/robots.txt -O /var/www/html/_default/robots.txt

###  重启 Apache

    service httpd restart

## 安装 MySQL 数据库

    yum install -y mysql-server
    service mysqld start
    chkconfig mysqld on

### 增强 MySQL 的安全性

    /usr/bin/mysql_secure_installation

## 安装 PHP

    yum install -y php php-pear php-mysql php-gd php-mbstring php-mcrypt

## 重启 Apache

    service httpd restart

## 重启服务器

    reboot

