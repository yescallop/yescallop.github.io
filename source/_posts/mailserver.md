---
title: 在树莓派上搭建邮件服务器 Postfix + Dovecot + SASL
date: 2016-07-02 20:54:14
updated: 2021-11-04 17:21:00
categories: 笔记
tags: Linux
---

最近入手了一台树莓派 3B，要把 ECS 上的各种服务器全部迁移到上面，配置邮件服务器的方法有些麻烦，所以在此写一篇教程分享给大家供借鉴。

<!-- more -->

> 更新 2017-04-29: 无法发送邮件，很久没有解决
> 更新 2017-08-23: 发件问题已修复

## 准备环境

**Raspberry Pi 3 Model B**，运行 **Raspbian** 最新版本

## 开始搭建

首先通过 apt-get 获取 Postfix，Dovecot 和 SASL

我个人认为 IMAP 比 POP3 好，所以在此没有配置 POP3，如有需要请加上 dovecot-pop3d

`sudo apt-get install postfix dovecot-imapd sasl2-bin`

### 配置 Postfix 和 SASL

下面部分内容借鉴了 Debian 官方 Wiki 的文章 [PostfixAndSASL](https://wiki.debian.org/PostfixAndSASL)

1. 创建一个文件 /etc/postfix/sasl/smtpd.conf，内容如下

    ```text /etc/postfix/sasl/smtpd.conf
    pwcheck_method: saslauthd
    mech_list: PLAIN LOGIN
    ```

2. 复制 /etc/default/saslauthd 到 /etc/default/saslauthd-postfix，修改内容如下

    ```text /etc/default/saslauthd-postfix
    START=yes
    DESC="SASL Auth. Daemon for Postfix"
    NAME="saslauthd-postf" # 最长为 15 字符
    OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"
    ```

3. 创建 Postfix 改变根目录所需的子目录，命令：

    `sudo dpkg-statoverride --add root sasl 710 /var/spool/postfix/var/run/saslauthd`

4. 将 postfix 用户加入组 sasl

    `sudo adduser postfix sasl`

5. 重启 saslauthd 服务

    `sudo service saslauthd restart`

6. 编辑 Postfix 配置文件 /etc/postfix/main.cf，以我的域名 yescallop.cn 为例，修改内容如下

    ```text /etc/postfix/main.cf
    myhostname = yescallop.cn
    myorigin = $myhostname
    mydestination = $myhostname, localhost
    smtpd_sasl_local_domain = $myhostname
    smtpd_sasl_auth_enable = yes
    broken_sasl_auth_clients = yes
    smtpd_sasl_security_options = noanonymous
    smtpd_recipient_restrictions = permit_sasl_authenticated, reject_unauth_destination
    ```

7. 重启 Postfix （重载配置是不够的）

    `sudo service postfix restart`

然后就可以使用 telnet 连接 25 端口测试了

### 配置 Dovecot

1. 编辑 Dovecot 配置文件 /etc/dovecot/dovecot.conf，修改内容如下：

    ```text /etc/dovecot/dovecot.conf
    listen = *
    ```

2. 编辑认证配置文件 /etc/dovecot/conf.d/10-auth.conf，修改内容如下：

    ```text /etc/dovecot/conf.d/10-auth.conf
    disable_plaintext_auth = no
    auth_mechanisms = plain login
    ```

3. 编辑 IMAP 配置文件 /etc/dovecot/conf.d/20-imap.conf，修改内容如下：

    ```text /etc/dovecot/conf.d/20-imap.conf
    mail_max_userip_connections = 100 # 此处可自定义，不要太小，容易导致连接被拒绝
    ```

4. 重启 Dovecot

    `sudo service dovecot restart`

可以使用 telnet 连接 143 端口测试

最后可以添加 Postfix，Dovecot 和 SASL 到启动组

```sh Commands
sudo update-rc.d postfix defaults
sudo update-rc.d dovecot defaults
sudo update-rc.d saslauthd defaults
```

**All Done!**
