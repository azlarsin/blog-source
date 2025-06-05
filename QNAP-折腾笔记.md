title: QNAP 折腾笔记
author: azlar
date: '2023-02-12 19:46:19'
tags: [qnap, ssh, 无法登录, 威联通, nas, telnet, raid1]

---
[TOC]
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

## crontab 配置被清除/丢失/清空
稳定运行了几年的 crontab，某天发现配置已被清空成默认：

```shell
[admin@Home-Nas .@backup_config]# crontab -l
9 9,21 * * * /sbin/notify_update --nc 1>/dev/null 2>&1
0 3 * * 0 /etc/init.d/idmap.sh dump
0-59/20 3 * * * /sbin/adjust_time
0 1 * * * /etc/init.d/flush_memory.sh >/dev/null 2>&1
0 4 * * * /sbin/hwclock -s
0 3 * * * /sbin/clean_reset_pwd
0-59/15 * * * * /etc/init.d/nss2_dusg.sh
10 15 * * * /usr/bin/power_clean -c 2>/dev/null
0 2 * * * /sbin/qfstrim
0-59/10 * * * * /etc/init.d/storage_usage.sh
30 3 * * * /sbin/notice_log_tool -v -R
0 0 * * * /sbin/user_cmd -C 1>/dev/null 2>&1
0 3 * * * /etc/init.d/wcl_backup.sh
30 * * * * /etc/init.d/wcl_check.sh
30 7 * * * /sbin/clean_upload_file
*/10 * * * * /sbin/config_cache_util 0
55 6 * * * /share/CACHEDEV1_DATA/.qpkg/HybridBackup/rr2/scripts/insight/insight.sh -runall >/dev/null 2>&1
9 2 * * * /bin/sh /etc/init.d/disk_data_collection.sh
0 8 * * 1 /etc/init.d/poweroff
55 19 * * 6 /etc/init.d/startup
* * * * * /var/cache/netmgr/lock_timer.sh
0 12 * * * /mnt/ext/opt/LicenseCenter/bin/qlicense_tool local_check
0 0 * * * /usr/local/sbin/qulog-archive local_event retention 0 >/dev/null 2>&1
0 1 * * * /usr/local/sbin/qulog-archive remote_event retention 0 >/dev/null 2>&1
0 2 * * * /usr/local/sbin/qulog-archive local_access retention 0 >/dev/null 2>&1
0 3 * * * /usr/local/sbin/qulog-archive remote_access retention 0 >/dev/null 2>&1
0 4 * * * (source /etc/init.d/qulog.sh dummy; check_mariadb) >/dev/null 2>&1
50 7 * * * /sbin/qpkg_cli --check_license 0 > /dev/null 2>/dev/null
0 4 * * * /etc/init.d/wsd.sh restart
0 10 * * * /sbin/vs_refresh
4 3 * * 3 /etc/init.d/backup_conf.sh
50 20 * * * /mnt/ext/opt/QcloudSSLCertificate/bin/ssl_agent_cli
0 0 * * * /usr/local/sbin/qsh nc.archive >/dev/null 2>&1
0 1 * * * (source /etc/init.d/nc.sh dummy; check_mariadb) >/dev/null 2>&1
35 7 * * * /sbin/qsyncsrv_util -c  > /dev/null 2>/dev/null
0 0 * * * /sbin/qsyncsrv_tool --fix  > /dev/null 2>/dev/null
* 4 * * * /usr/sbin/logrotate /etc/config/mc_logr.conf
* 4 * * * /usr/sbin/logrotate /etc/config/mariadb_mc.logr
0 2 * * 0 /usr/local/medialibrary/bin/mymediadbcmd dbbackup >/dev/null 2>&1
0 0 * * * /usr/local/medialibrary/bin/mlidbcli clear --only-unused-thumb >/dev/null 2>&1
[admin@Home-Nas .@backup_config]#
```

一番搜索，貌似是[传统艺能](https://forum.qnap.com/viewtopic.php?t=173307&sid=c62a1d3664f584994aac643081a8ee9d)了，不过比较好奇，为什么运行了这么久才出问题。

由于不记得某些配置（流水账未记录的某些脚本）如何配的了，于是开始找寻文件备份方案。

### 方案

#### 定位 `.@backup_config`
```shell
# 系统盘
[admin@Home-Nas .@backup_config]# pwd
/share/CACHEDEV1_DATA/.@backup_config
[admin@Home-Nas .@backup_config]# ll
total 168M
drwxr-xr-x  2 admin administrators 4.0K 2025-02-09 02:46 ./
drwxrwxrwx 45 admin administrators 4.0K 2025-02-09 02:29 ../
-rw-r--r--  1 admin administrators  17M 2025-01-29 03:04 0_20250129_0304.bak
-rw-r--r--  1 admin administrators  17M 2025-01-22 03:04 1_20250122_0304.bak
-rw-r--r--  1 admin administrators  17M 2025-01-15 03:04 2_20250115_0304.bak
-rw-r--r--  1 admin administrators  17M 2025-01-08 03:04 3_20250108_0304.bak
-rw-r--r--  1 admin administrators  17M 2025-01-01 03:04 4_20250101_0304.bak
-rw-r--r--  1 admin administrators  17M 2024-12-25 03:04 5_20241225_0304.bak
-rw-r--r--  1 admin administrators  17M 2024-12-18 03:04 6_20241218_0304.bak
-rw-r--r--  1 admin administrators  17M 2024-12-11 03:04 7_20241211_0304.bak
-rw-r--r--  1 admin administrators  17M 2024-12-04 03:04 8_20241204_0304.bak
-rw-r--r--  1 admin administrators  17M 2024-11-29 13:04 9_20241129_1304.bak
[admin@Home-Nas .@backup_config]# hexdump -C 0_20250129_0304.bak | head
00000000  50 4b 03 04 14 00 09 00  08 00 82 18 3d 5a 9f 57  |PK..........=Z.W|
00000010  1b 3a c8 3f 0f 01 54 1f  11 01 3b 00 1c 00 73 68  |.:.?..T...;...sh|
00000020  61 72 65 2f 43 41 43 48  45 44 45 56 31 5f 44 41  |are/CACHEDEV1_DA|
00000030  54 41 2f 2e 40 62 61 63  6b 75 70 5f 63 6f 6e 66  |TA/.@backup_conf|
00000040  69 67 2f 30 5f 32 30 32  35 30 31 32 39 5f 30 33  |ig/0_20250129_03|
00000050  30 34 2e 74 61 72 2e 67  7a 55 54 09 00 03 24 2a  |04.tar.gzUT...$*|
00000060  99 67 20 2a 99 67 75 78  0b 00 01 04 00 00 00 00  |.g *.gux........|
00000070  04 00 00 00 00 13 e0 5b  1d 85 d0 a5 ae e3 c3 06  |.......[........|
00000080  78 84 72 ee 27 4a bf e3  2a 67 53 34 ef 68 ea b2  |x.r.'J..*gS4.h..|
00000090  47 01 2f aa eb ba 02 12  ba 61 1c 66 9c 28 e7 94  |G./......a.f.(..|
[admin@Home-Nas .@backup_config]#
```

#### unzip
找到是 zip 格式的文件，尝试 unzip 解析。

```shell
[admin@Home-Nas .@backup_config]# unzip 0_20250129_0304.bak
Archive:  0_20250129_0304.bak
[0_20250129_0304.bak] share/CACHEDEV1_DATA/.@backup_config/0_20250129_0304.tar.gz password:
   skipping: share/CACHEDEV1_DATA/.@backup_config/0_20250129_0304.tar.gz  incorrect password
[admin@Home-Nas .@backup_config]# unzip 0_20250129_0304.bak
Archive:  0_20250129_0304.bak
[0_20250129_0304.bak] share/CACHEDEV1_DATA/.@backup_config/0_20250129_0304.tar.gz password:
password incorrect--reenter:
password incorrect--reenter:
   skipping: share/CACHEDEV1_DATA/.@backup_config/0_20250129_0304.tar.gz  incorrect password
[admin@Home-Nas .@backup_config]#
```

密码不是 mac/admin/"admin" 密码，而是 `/sbin/gen_encstr` 这个神奇命令所输出的字符串，类似：`V2@wewqeqwewelqwjwewqeA==`

> source: 
> - https://community.qnap.com/t/shared-folders-not-shown-in-control-panel-after-fw-update/393/6
> https://www.reddit.com/r/qnap/comments/128q7lt/qnap_backup_configuration_password/


#### 结束

```shell
[admin@Home-Nas .@backup_config]# unzip -P "$(/sbin/gen_encstr)" 3_20250108_0304.bak
Archive:  3_20250108_0304.bak
  inflating: share/CACHEDEV1_DATA/.@backup_config/0_20250108_0304.tar.gz
[admin@Home-Nas .@backup_config]# ls
0_20250129_0304.bak  2_20250115_0304.bak  4_20250101_0304.bak  6_20241218_0304.bak  8_20241204_0304.bak  share/
1_20250122_0304.bak  3_20250108_0304.bak  5_20241225_0304.bak  7_20241211_0304.bak  9_20241129_1304.bak
[admin@Home-Nas .@backup_config]# cd share/CACHEDEV1_DATA/.\@backup_config/
[admin@Home-Nas .@backup_config]# ls
0_20250108_0304.tar.gz
[admin@Home-Nas .@backup_config]# tar xzf *.tar.gz
[admin@Home-Nas .@backup_config]# ll
total 17M
drwxr-xr-x 4 admin administrators 4.0K 2025-02-09 02:57 ./
drwxr-xr-x 3 admin administrators 4.0K 2025-02-09 02:57 ../
-rw-r--r-- 1 admin administrators  17M 2025-01-08 03:04 0_20250108_0304.tar.gz
drwxr-xr-x 4 admin administrators 4.0K 2025-02-09 02:57 etc/
drwxr-xr-x 3 admin administrators 4.0K 2025-02-09 02:57 share/
[admin@Home-Nas .@backup_config]# cat /etc/config/crontab
9 9,21 * * * /sbin/notify_update --nc 1>/dev/null 2>&1
0 3 * * 0 /etc/init.d/idmap.sh dump
0-59/20 3 * * * /sbin/adjust_time
0 1 * * * /etc/init.d/flush_memory.sh >/dev/null 2>&1
0 4 * * * /sbin/hwclock -s

# 结束
```


### 思考
本来写了个脚本监控 crontab 丢失，写完发现还是需放到 crontab 内运行，这样仍然有丢失风险。

所以需换个方式来触发 crontab 检查/补全 逻辑。




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
