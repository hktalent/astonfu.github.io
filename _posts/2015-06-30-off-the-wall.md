---
layout: post
title: Off The Wall
---

对，我喜欢Vans！无拘无束，见沟飞，见墙翻。

只为科学上网！

没有VPS又不想花钱买VPN的话用goAgent，有VPS的话可以架VPN，也可以使用隧道接口然后在使用时反向代理。

# [goAgent](https://github.com/goagent/goagent)

虽然免费，但不稳定，总要升级。

# [ShadowSocks](https://github.com/shadowsocks/shadowsocks)


## Server搭建

Debian/Ubuntu

```
apt-get install python-pip
pip install shadowsocks
```

### 打开服务

```
ssserver -p 443 -k password -m aes-256-cfb
```

### 在后台运行：

```
sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start
```

### 停止服务：

```
sudo ssserver -d stop
```

### 查看Log：

```
sudo less /var/log/shadowsocks.log
```

### 自动启动
新建 /etc/init.d/shadowsocks 文件，内容如下：

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          shadowsocks
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: start shadowsocks
# Description:       start shadowsocks
### END INIT INFO

start(){
    ssserver -c /etc/shadowsocks.config -d start
}

stop(){
    ssserver -c /etc/shadowsocks.config -d stop
}

case "$1" in
start)
    start
    ;;
stop)
    stop
    ;;
reload)
     stop
     start
     ;;
*)
    echo "Usage: $0 {start|reload|stop}"
    exit 1
    ;;
esac
```

然后

```bash
# 增加这个文件的可执行权限
sudo chmod +x /etc/init.d/shadowsocks

# 在 rc.d 中新增，添加到开机启动中
sudo update-rc.d shadowsocks defaults

# 可以在shell中直接运行
sudo service shadowsocks {start|reload|stop}
```


参考：
http://blog.xavierskip.com/2015-02-02-shadowsocks-init/

### 命令行也proxy
使用[proxychains-ng](https://github.com/rofl0r/proxychains-ng)

配置文件$(HOME)/.proxychains/proxychains.conf：

```
strict_chain
proxy_dns
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000
localnet 127.0.0.0/255.0.0.0
quiet_mode

[ProxyList]
socks5  127.0.0.1 1080
```

单独命令使用：

```
proxychains4 curl https://www.twitter.com/
proxychains4 git push origin master
```

proxify bash 使用:

```
proxychains4 bash
curl https://www.twitter.com/
git push origin master
```

### 中继
参考 https://github.com/shadowsocks/shadowsocks/wiki/Setup-a-Shadowsocks-relay

安装 HAProxy 后编辑 /etc/haproxy/haproxy.cfg 文件：

```bash
global
        ulimit-n  51200

defaults
        log global
        mode    tcp
        option  dontlognull
        contimeout 1000
        clitimeout 150000
        srvtimeout 150000

frontend ss-in
        bind *:8388
        default_backend ss-out

backend ss-out
        server server1 remote-server:2222 maxconn 20480

```

remote-server 是运行 SS 的 Server，把它映射到了 本地的 8388 端口。

主要这里主要把 mode 设置为 **tcp**，我的默认就是 http，当然还有其对应的 log 什么的。

## Linux/MacOS Client 使用

1.首先也安装ss：pip install shadowsocks

2.然后执行下面的命令，不用装什么客户端：

```
sslocal -s IP -p PORT -b 127.0.0.1 -l 1080 -k PASSWORD -t 600 -m aes-256-cfb

```

3.在Chrome下用[SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega)设置即可。

## iOS 使用

去下载官方应用，[iTunes走起](https://itunes.apple.com/us/app/shadowsocks/id665729974?ls=1&mt=8)。

### 本地全局PAC代理

但有以下限制：

* 需要WiFi联网。
* 只能用几分钟。因为iOS的限制，它不能在后台一直运行，所以需要不时的来看看。

设置代理：

* 打开iOS的设置 -> Wi-Fi -> i 图标（在当前所连WiFi的后面） -> HTTP 代理
* 选择自动，输入http://127.0.0.1:8090/proxy.pac
* 返回

# SSH

下面建立了一条绑定本地**7070**端口的静默隧道，-p 表示server shh的端口。

```
ssh -qTfnN -D 7070 user@server.address -p 22
```

相比ShadowSocks，SSH也是使用socket5来建立隧道，更简单，但是不如SS方便手机使用，而且比SS稍慢。

参考：

* http://www.williamlong.info/archives/4121.html
* http://vpsnews.cc/shadowsocks-ssh%E5%8C%BA%E5%88%AB/

## ssh chains
local <--> A <--> B

最傻瓜的方法：

```bash
local$ ssh user@A
A$ ssh user@B
```

升级：

```bash
local$ ssh -A -t user@A ssh user@B
```

更好的方法，使用 'ProxyCommand' 设置 SSH config 文件，同时需要在跳板上安装 netcat，修改 ~/.ssh/config 文件：

```bash
Host A
  HostName aaaa.com

Host B
  ProxyCommand ssh -q A nc -q0 B 22
```


# VPN
VPN用着很方便，还能发放帐号呢。

VPN目前有PPTP、L2TP、IPSec 和 IKEv2，推荐IKEv2.

## IKEv2

IKEv2 (Internet Key Exchange v2) 是一款新版的安全协议。
它有着一个独有的特性，就是在移动客户端改变网络状态的情况下，
加密通讯不会因此中断，因此特别适应便携设备的安全。

IKEv2 是 IKE(v1) 的全新改良版，传输效率大大超于 PPTP 和 L2TP，使用公钥证书和密码等多重认证，
支持硬件加速，保持了高效的传输效率。

如果出门在外，(手机) 网络环境 (IP 地址) 经常变动的情况下， IKEv2 是唯一使用 “MOBIKE” – Mobility and Multihoming 技术来达到加密通讯不中断目的的。
所以用户再也无需考虑手动开启或者关闭 VPN，或者担心因网络环境变化而导致的 VPN 连接中断了。

而且这里有方便一键安装脚本 —— [one-key-ikev2](https://github.com/quericy/one-key-ikev2-vpn).

证书可以用 acme.sh 来申请，参考[https://quericy.me/blog/860/]并写一个更新证书的脚本 update_ipsec_cert：

```bash
#! /bin/bash
cert_file="/home/user/.acme.sh/yourdomain/yourdomain.cer"
key_file="/home/user/.acme.sh/yourdomain/yourdomain.key"

sudo cp -f $cert_file /usr/local/etc/ipsec.d/certs/server.cert.pem
sudo cp -f $key_file /usr/local/etc/ipsec.d/private/server.pem
sudo cp -f $cert_file /usr/local/etc/ipsec.d/certs/client.cert.pem
sudo cp -f $key_file /usr/local/etc/ipsec.d/private/client.pem
sudo /usr/local/sbin/ipsec restart
```

并 chmod +x 让其可执行。

在更新 Nginx 证书的时候顺便更新它的：

```bash
acme.sh --installcert -d www.your-app.com \
               --keypath       /home/ubuntu/www/ssl/www.your-app.com.key  \
               --fullchainpath /home/ubuntu/www/ssl/www.your-app.com.key.pem \
               --reloadcmd     "sudo service nginx force-reload && sudo /home/ubuntu/bin/update_ipsec_cert"
```


### 客户端配置说明

- 连接的服务器地址和证书保持一致,即取决于签发证书ca.cert.pem时使用的是ip还是域名;

- Android/iOS/OSX 可使用ikeV1,认证方式为用户名+密码+预共享密钥(PSK);

- iOS/OSX/Windows7+/WindowsPhone8.1+/Linux 均可使用IkeV2,认证方式为用户名+密码。使用SSL证书则无需导入证书；使用自签名证书则需要先导入证书才能连接,可将ca.cert.pem更改后缀名作为邮件附件发送给客户端,手机端也可通过浏览器导入,其中:

- iOS/OSX 的远程ID和服务器地址保持一致,用户鉴定选择"用户名".如果通过浏览器导入,将证书放在可访问的远程外链上,并在系统浏览器(Safari)中访问外链地址.OSX证书需要设置为始终信任;
Windows PC 系统导入证书需要导入到"本地计算机"的"受信任的根证书颁发机构",以"当前用户"的导入方式是无效的.推荐运行mmc添加本地计算机的证书管理单元来操作;

### 其他

#### ipsec启动问题

服务器重启后默认ipsec不会自启动，请命令手动开启,或添加/usr/local/sbin/ipsec start到自启动脚本文件中(如rc.local等)。

Ubuntu 已经在 /etc/init 里创建了 strongswan-start.conf 来自动启动了。

#### ipsec常用指令

```bash
ipsec start   #启动服务
ipsec stop    #关闭服务
ipsec restart #重启服务
ipsec reload  #重新读取
ipsec status  #查看状态
ipsec --help  #查看帮助
```

## PPTP

老技术了，虽然还经常被用，但已经被慢慢淘汰，瞧iOS 10就不支持了。

### 1. 安装PPTPD

```
apt-get install pptpd
```

### 2. 编辑VPN的接口IP地址

打开**/etc/pptpd.conf**，找到：

```
#localip 192.168.0.1
#remoteip 192.168.0.234-238,192.168.0.245
```

去掉"#"。

### 3. 设置DNS地址
打开**/etc/ppp/pptpd-options**，找到：

```
#ms-dns 10.0.0.1
#ms-dns 10.0.0.2
```

变为google的DNS地址：

```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

当然也可以是其它公开可用的，如OpenDNS: 208.67.222.222 & 208.67.220.220。
(Tips: You can also copy the above codes and paste it under the original ones.)

### 4. 添加VPN帐号
打开**/etc/ppp/chap-secrets**，在文件最后加入帐号，格式如下，用TAP街隔开：

username   pptpd   password   *

比如：

freenuts   pptpd   123456   *

### 5. 设置转发IPv4
打开**/etc/sysctl.conf**，找到：

```
#net.ipv4.ip_forward=1
```

去掉签名注释"#"。

### 6. 更新转发生效
输入下面命令：

```
sysctl -p
```

如果正确的话会返回：

net.ipv4.ip_forward = 1

### 6. 设置防火墙允许路由

```
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
```

### 7. 重启PPTPD

```
/etc/init.d/pptpd restart
```

然后搜索如何使用PPTP（Point-to-Point Tunneling Protocol） VPN，简单设置即可使用了。

上帝用七天创造了世界，我们用七步走进了更广阔的世界，阿门！

参考：http://freenuts.com/how-to-set-up-a-vpn-in-a-vps/

## UFW防火墙配置

### 允许1723默认端口
```
sudo ufw allow 1723
```

### 在**/etc/default/ufw**里将DEFAULT_FORWARD_POLICY 变成 “ACCEPT”:

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

### 在**/etc/ufw/before.rules**文件里

#### 1. 在Don't delete these required lines上面加入

```
# NAT table rules
*nat

:POSTROUTING ACCEPT [0:0]
# Allow forward traffic to eth0
-A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE

# Process the NAT table rules
COMMIT

# Don't delete these required lines, otherwise there will be errors
...
```

#### 2. 在filter部分前面加入（其实位置都无所谓，只要在require后，COMMIT前即可）

```
# First, since we trust pptpd completely, I would accept all traffic to/from my pptpd. I added this lines at the beginning of the filter section.
-A ufw-before-input -i ppp+ -j ACCEPT
-A ufw-before-output -i ppp+ -j ACCEPT

# Additionally, I must forward traffic to/from my pptpd. These lines was also added after the above lines.
-A ufw-before-forward -s 192.168.0.0/24 -j ACCEPT
-A ufw-before-forward -d 192.168.0.0/24 -j ACCEPT
```

#### 3. 在drop INVALID packets前面加上：

```
-A ufw-before-input -p 47 -j ACCEPT
-A ufw-before-output -p 47 -j ACCEPT

# drop INVALID packets (logs these in loglevel medium and higher)
...
```

#### 参考：

* http://www.cviorel.com/2009/02/09/how-to-set-up-a-vpn-server-on-ubuntu/
* http://askubuntu.com/questions/119534/easiest-way-to-setup-ubuntu-as-a-vpn-server
* http://ubuntuforums.org/showthread.php?t=1113911
* https://i.ssvpn.me/blog/id=5
* http://sshmenu.sourceforge.net/articles/transparent-mulithop.html
* http://kaywu.xyz/2016/06/19/Shadowsocks-HAProxy/

# 关于VPS

我目前用的是DigitalOcean的最低版本（5美元/月），这里有推荐链接，使用可以多得10美元：[https://www.digitalocean.com/?refcode=3d496b50e388](https://www.digitalocean.com/?refcode=3d496b50e388)
