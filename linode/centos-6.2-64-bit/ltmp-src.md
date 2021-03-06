# Linode CentOS 6.2 64 位 LTMP 编译安装教程（未完成）

成功安装后的网站根目录和一些配置文件的路径

* Apache 网站根目录 `/data/nginx/www/public`
* 配置文件
    * Tengine `/data/nginx/conf`
    * MariaDB `/data/mysql`
    * PHP `/data/php`

## 设置反向域名解析，添加局域网 IP，再重启服务器

1. 在 DNS 中添加一条指向服务器 IP 的 A 记录，例如：`host-1.project-name.com => 12.34.56.78`
2. 在 Linode 后台中，修改服务器 IP 的反向解析为刚才设置的域名
3. 在 Linode 后台中，点击分配一个局域网 IP
4. 重启服务器

## 设置主机名

    echo 'HOSTNAME=project-name-1' >> /etc/sysconfig/network
    hostname 'project-name-1'

## 编辑 /etc/hosts

    127.0.0.1       localhost.localdomain   localhost
    12.34.56.78     host-1.project-name.com project-name-1

## 设置时区

    ln -sf /usr/share/zoneinfo/UTC /etc/localtime

## 安装 EPEL 和 Remi，并安装软件更新

    rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    
    rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
    wget https://raw.github.com/a25ce1/server-setup/master/etc/remi.repo -O /etc/yum.repos.d/remi.repo
    
    yum update -y

## 创建防火墙

    nano /etc/iptables.firewall.rules

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

## 安装和配置 Fail2Ban

    yum install fail2ban
    
    wget https://raw.github.com/a25ce1/server-setup/master/etc/jail.local -O /etc/fail2ban/jail.local
    
    chkconfig fail2ban on

## 安装 Tengine 服务器

    yum -y install gcc make
    
    yum -y install zlib-devel pcre-devel openssl-devel
    
    cd /usr/local/src
    wget http://tengine.taobao.org/download/tengine-1.5.1.tar.gz
    tar zxvf tengine-1.5.1.tar.gz
    cd tengine-1.5.1
    ./configure \
        --prefix=/data/nginx \
        --with-http_gzip_static_module \
        --with-http_concat_module
    make
    make install
    
    wget https://raw.github.com/a25ce1/server-setup/master/etc/nginx -O /etc/init.d/nginx
    chmod +x /etc/init.d/nginx
    chkconfig nginx on

