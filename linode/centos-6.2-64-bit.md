1. 设置反向域名解析、局域网 IP，然后重启服务器
----------------------------------------------

2. 设置主机名
-------------

```
echo 'HOSTNAME=chenhua-1' >> /etc/sysconfig/network
hostname 'chenhua-1'
```

3. 编辑 /etc/hosts
------------------

```
127.0.0.1       localhost.localdomain   localhost
106.186.119.111 host-1.chenhua.me       chenhua-1
```

4. 设置时区
-----------

```
ln -sf /usr/share/zoneinfo/UTC /etc/localtime
```

5. 安装 EPEL 和 Remi，并安装软件更新
------------------------------------

```
rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
wget https://raw.github.com/a25ce1/server-setup/master/etc/remi.repo -O /etc/yum.repos.d/remi.repo

yum update -y
```

6. 创建新用户
-------------



7. 创建防火墙
-------------

创建并编辑文件 `/etc/iptables.firewall.rules`

```
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
```

应用防火墙规则

```
iptables-restore < /etc/iptables.firewall.rules
```

保存当前防火墙规则

```
/sbin/service iptables save
```

8. 安装和配置 Fail2Ban
----------------------

