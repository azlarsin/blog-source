title: QNAP 折腾笔记
author: azlar
date: '2023-02-12 19:46:19'
tags: [qnap, ssh, 无法登录, 威联通, nas, telnet, raid1]

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
2. crontab 任务掉了


## RAID1 降级为单盘
[https://forum.qnap.com/viewtopic.php?f=25&t=150513](https://forum.qnap.com/viewtopic.php?f=25&t=150513)
### 1. 关机拔盘
### 2. 开机
提示，降级中。ssh 进入：

```shell
# mdadm --query -- detail /dev/md1      # 找到对应的 /dec/md{x}

# mdadm --grow /dev/md2 --raid-devices=1 -- force # 降级
```

这里，已经可以看到成为单盘里。（下次遇到，跳过 3，直接重启试试）

### 3. /etc/config/raid.conf
引用：
> ```
>    配置文件路径："/etc/config/raid.conf"，找到目标raid组；
>    删掉："scrubstatus, eventskipped,eventcompleted,degradedcnt,data_0"开头的项及" [Remove] "；
>    修改：
>        "data_1 = 2，xxx(序列号)" 改为 "data_0 = 1，xxx(序列号)"
>
>       chunkSize的值改为0
>
>        readAhead的值改为0
>        databitmap的值改为1
>        ```
>

**重要，改完这一步后，重启直接报错：磁盘损坏只读了**

### 4. 快照恢复
本来改完 3，系统盘就 gg 了，还好找到一份快照，直接恢复成功了 ![](//blog.azlar.cc/images/emoji/facepalm.jpg)

然后就可以愉快的做 ssd 缓存了。


## crontab
`crontab -e` 重启后各种丢失。

```shell
# vi /etc/config/crontab
# crontab /etc/config/crontab
# /etc/init.d/crond.sh restart
# crontab -l            # 顺序可能错乱
```


## LetsEncrypt ssl 证书上传不上
[原因](https://forum.qnap.com/viewtopic.php?t=164839) 是网页只能接 RSA2048，而 [LetsEncrypt 已经默认生成 ECDSA](https://github.com/acmesh-official/acme.sh/issues/2350) 了。

### 方案
```shell
# add -k 2048
./acme.sh --force --issue -k 2048 --dns dns_cf -d
```


## emby ssl
```shell
openssl pkcs12 -export -out /tmp/wildcard.pfx -inkey privkey.pem -in cert.pem -certfile chain.pem
```

## openssl 传递

需求：远端自动 renew ssl 证书，nas wget 后，自动替换。

### openssl 版本问题
NAS 内置 openssl 1.1.1，远端使用 1.0.0，所以不可使用：

~~`openssl enc -e -des3 -a -salt -k "xxx"`~~

NAS 解析会报错：

```shell
# openssl -d xxx
18+1 records in
18+1 records out
9718 bytes (9.5KB) copied, 0.000038 seconds, 243.9MB/s
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bad decrypt
```

#### 解决
升级远端(centos) openssl 与 nas 一致。

```shell
# remote centos
cd /root/.acme.sh/ && tar -czPf - DOMAIN_ECC/ | openssl aes-256-cbc -a -salt -pbkdf2 -k "XXX" | dd of="/data/static/FILE"

# nas
dd if=./FILE |openssl aes-256-cbc -d -a -pbkdf2 -k "XXX" | tar -zxPf -
```


## 替换 ssl 证书
### 参考 https://www.qnap.com/en/how-to/faq/article/how-to-replace-ssl-certificate-manually-on-ssh-console

```shell
cat domain.key domain.cer > /etc/stunnel/stunnel.pem

```


## ffmpeg
### 需求
合并 QVR 的视频为一个。按天 merge。

### 问题
1. 系统自带的 ffmpeg 版本比较旧，无法使用 acc 音频解码。导致无法合并。
2. ffmpeg 对路径中 空格的识别问题
 
自带的 ffmpeg
```shell
ffmpeg version 3.3.6 Copyright (c) 2000-2017 the FFmpeg developers
  built with gcc 4.9.2 (Debian 4.9.2-10)
  configuration: --enable-cross-compile --arch=i686 --target-os=linux --disable-yasm --disable-static --enable-shared --enable-gpl --enable-libmp3lame --disable-libx264 --enable-libsoxr --enable-version3 --enable-nonfree --disable-openssl --disable-decoder=ac3 --disable-decoder=ac3_fixed --disable-decoder=eac3 --disable-decoder=dca --disable-decoder=truehd --disable-encoder=ac3 --disable-encoder=ac3_fixed --disable-encoder=eac3 --disable-encoder=dca --disable-decoder=hevc --disable-decoder=hevc_cuvid --disable-encoder=hevc_nvenc --disable-encoder=nvenc_hevc --disable-decoder=h264 --disable-decoder=h264_cuvid --disable-encoder=libx264 --disable-encoder=libx264rgb --disable-encoder=h264_nvenc --disable-encoder=nvenc --disable-encoder=nvenc_h264 --disable-decoder=mpeg2video --disable-decoder=mpegvideo --disable-decoder=mpeg2_cuvid --disable-encoder=mpeg2video --disable-decoder=mpeg4 --disable-decoder=mpeg4_cuvid --disable-decoder=msmpeg4v1 --disable-decoder=msmpeg4v2 --disable-decoder=msmpeg4v3 --disable-encoder=mpeg4 --disable-encoder=msmpeg4v2 --disable-encoder=msmpeg4v3 --disable-decoder=mvc1 --disable-decoder=vc1 --disable-decoder=vc1_cuvid --disable-decoder=vc1image --disable-decoder=aac --disable-decoder=aac_fixed --disable-decoder=aac_latm --disable-encoder=aac --disable-decoder=on2avc --disable-encoder=ssa --disable-encoder=ass --disable-encoder=dvbsub --disable-encoder=dvdsub --disable-encoder=movtext --disable-encoder=srt --disable-encoder=subrip --disable-encoder=text --disable-encoder=webvtt --disable-encoder=xsub --disable-encoder=movtext --disable-decoder=ssa --disable-decoder=ass --disable-decoder=dvbsub --disable-decoder=dvdsub --disable-decoder=ccaption --disable-decoder=pgssub --disable-decoder=jacosub --disable-decoder=microdvd --disable-decoder=movtext --disable-decoder=mpl2 --disable-decoder=pjs --disable-decoder=realtext --disable-decoder=sami --disable-decoder=stl --disable-decoder=srt --disable-decoder=subrip --disable-decoder=subviewe --disable-decoder=subviewe --disable-decoder=text --disable-decoder=vplayer --disable-decoder=webvtt --disable-decoder=xsub --disable-decoder=ccaption --disable-decoder=movtext --disable-decoder=subviewer --disable-decoder=subviewer1 --extra-ldflags='-L/root/workspace/x86_64/ndk/LinkFS/usr/lib -L/root/workspace/x86_64/ndk/Model/TS-X53/build/RootFS/usr/local/medialibrary/lib -Wl,--rpath -Wl,/usr/local/medialibrary/lib' --extra-cflags='-I/root/workspace/x86_64/ndk/LinkFS/usr/include -I/root/workspace/x86_64/ndk/Model/TS-X53/build/RootFS/usr/local/medialibrary/include -D_GNU_SOURCE -DQNAP -fstack-protector-strong -fPIE' --extra-ldexeflags='-pie -Wl,-pie' --prefix=/root/workspace/x86_64/ndk/Model/TS-X53/build/RootFS/usr/local/medialibrary
  libavutil      55. 58.100 / 55. 58.100
  libavcodec     57. 89.100 / 57. 89.100
  libavformat    57. 71.100 / 57. 71.100
  libavdevice    57.  6.100 / 57.  6.100
  libavfilter     6. 82.100 /  6. 82.100
  libswscale      4.  6.100 /  4.  6.100
  libswresample   2.  7.100 /  2.  7.100
  libpostproc    54.  5.100 / 54.  5.100
Hyper fast Audio and Video encoder
usage: ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...
```

### 解决
1. 手动安装：https://www.qnapclub.eu/en/qpkg/379。

```shell
/opt/ffmpeg/ffmpeg -y -f concat -safe 0 -i ${TXT_FILE_PATH} -c:v copy ${TARGET_OUTPUT_FILE_PATH} & # nas
```


1. 封装为 `file '/path1 path2/file'` 即可。双引号不可工作。


## Qsync 启用后无法工作
报错：`文件同步已停止，用户家目录已被停用。`

### 解决
不得不吐槽，只有一个报错，没有任何引导，qsync-client 也连接不上。必须去设置里修改：


> 找到『控制台』打开，依次打开『权限』-『用户』，点击『用户家目录』，将『启用所有用户的家目录』选项打勾，应用即可。
