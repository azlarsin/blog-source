title: QNAP 折腾笔记
author: azlar
date: '2023-02-12 19:46:19'
tags: [qnap, ssh, 无法登录, 威联通, nas, telnet]

---

<!-- desc -->

## Qnap NAS ssh 无法登录
用了几天，发现 ssh 挂了，无法登录。
### 问题
1. ssh 登不上；无论怎么设置 允许 ssh 都无效。
2. telnet 登不上（忘记初次 MAC 地址）

### 解决方案
1. [开机情况下捅 reset 3秒](https://docs.qnap.com/operating-system/qts/5.0.x/en-us/system-reset-and-restore-to-factory-default-1D42CBD6.html)。
2. telnet 连上机器后查看 `/etc/init.d/login.sh`：

```shell
telnet [ip] [port]
>> login: admin
>> pwd: [first MAC]
>> 
#  /etc/init.d/login.sh start
Starting sshd service: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0755 for '/etc/ssh/ssh_host_rsa_key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Unable to load host key "/etc/ssh/ssh_host_rsa_key": bad permissions
Unable to load host key: /etc/ssh/ssh_host_rsa_key
```

`Permissions 0755 for '/etc/ssh/ssh_host_rsa_key' are too open` => 修改权限：

```shell
[~] # chmod 400 /etc/ssh/ssh_host_rsa_key
[~] # ll -h /etc/ssh/ssh_host_rsa_key
-r-------- 1 admin administrators 2.6K 2023-02-07 10:13 /etc/ssh/ssh_host_rsa_key
[~] # /usr/sbin/sshd -f /etc/config/ssh/sshd_config -p 22
```

不明白系统是咋工作的，或者我干了什么操作（最可疑的是 scp），为啥会报这个。

#### 后果
1. nas 登录端口强制恢复到 8080
