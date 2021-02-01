---
title: 环境安装fail2ban
date: 2018-05-25 20:32:34
tags: ["环境搭建"]
categories: ["环境搭建"]
---

&emsp;&emsp;呵呵，一个月依赖，我的梯子总是时不时的会宕机，然后，就重新启动。好几次都是这样，按理说，centos系统应该是很稳定的，不至于频繁宕机，然后看资源也是充足的，除了自己搭建的一个梯子，并没有其他应用部署在这台机器上，看了一下服务器的日志信息，系统的/var/log/下面的secure文件似乎有点大，
<!--more-->
![img](fail2ban开始.png)
然后检查今天的secure,有惊喜发现，`O(∩_∩)O哈哈~`。
命令`grep 'Failed password' secure | tail -20f`
![img](fail2ban暴力ip地址.png)
然后就是这样，不同的ip用暴力破解的方式，以不同的登陆名及密码，不断的登陆，也就是暴力破解。找到原因了，那么就是解决方案，问了前公司群里的大牛，一句，‘fail2ban 了解一下’，呵呵，开搞。
#### 什么是fail2ban？
&emsp;&emsp;Fail2Ban 是一款入侵防御软件，可以保护服务器免受暴力攻击。 它是用 Python 编程语言编写的。 Fail2Ban 基于auth 日志文件工作，默认情况下它会扫描所有 auth 日志文件，如 /var/log/auth.log、/var/log/apache/access.log 等，并禁止带有恶意标志的IP，比如密码失败太多，寻找漏洞等等标志。
&emsp;&emsp;通常，Fail2Ban 用于更新防火墙规则，用于在指定的时间内拒绝 IP 地址。 它也会发送邮件通知。 Fail2Ban 为各种服务提供了许多过滤器，如 ssh、apache、nginx、squid、named、mysql、nagios 等。
&emsp;&emsp;Fail2Ban 能够降低错误认证尝试的速度，但是它不能消除弱认证带来的风险。 这只是服务器防止暴力攻击的安全手段之一。

#### fail2ban centos 安装
`yum -y install epel-release.noarch`
`yum install fail2ban`

&emsp;&emsp;主配置文件为jail.conf，这个文件官方注释为不建议修改，需要新建并编辑jail.local 文件，添加如下规则
配置规则
```
[DEFAULT]
# 屏蔽时常1天 设置为 -1 永久屏蔽
bantime = 86400
# 最大失败次数3次
maxretry = 3  
[sshd]
enabled = true
```
启动fail2ban服务
`systemctl restart fail2ban`

启动Fail2ban并设置为开机启动.
`systemctl enable fail2ban`
`systemctl start fail2ban`

查看拦截日志
`fail2ban-client status sshd`
添加白名单
`fail2ban-client set sshd addignoreip IP地址`
删除白名单
`fail2ban-client set sshd delignoreip IP地址`
查看Fail2ban日志
`tail /var/log/fail2ban.log`  
查看被禁用的ip
`ipset list`
删除被禁用的ip地址
`ipset del fail2ban-sshd 172.20.20.51 -exist`
`fail2ban-client set ssh unbanip 172.20.20.51`

&emsp;&emsp;配置了file2ban之后，看着屏蔽了的ip，一个字，爽，
![img](fail2ban屏蔽的ip.png)
&emsp;&emsp;嘿嘿，你攻击我，觉得我会放过你吗？不可能，闲的时候，研究一下反向追踪。。。


参考资料：
https://www.zcfy.cc/article/how-to-protect-server-against-brute-force-attacks-with-fail2ban-on-linux
https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-centos-7
