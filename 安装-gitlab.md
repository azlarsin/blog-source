title: 安装 gitlab
author: azlar
date: '2017-03-01 10:24:46'
tags: [gitlab, install gitlab]

---

小记。
<!-- desc -->

# 安装 gitlab

## 步骤
### 更换 yum 源
```shell	
$ cd /etc/yum.repos.d
$ wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

### 安装必要包
```shell
$ yum -y install libicu-devel patch gcc-c++ readline-devel zlib-devel libffi-devel openssl-devel make autoconf automake libtool bison libxml2-devel libxslt-devel libyaml-devel zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
```

### 更新 git
```shell
$ wget -O git-src.zip https://github.com/git/git/archive/master.zip
$ unzip git-src.zip
$ cd git-src
$ make all
$ make install
```

可能会遇到 `libiconv` 未定义的问题，手动安装后：

```shell
$ make configure
$ ./configure --prefix=/usr/local
$ make all doc
$ sudo make install install-doc install-html
```

### 安装 ruby
```shell
$ cd /usr/local/src
$ mkdir ruby && cd ruby
$ wget -0 ftp://ftp.ruby-lang.org/pub/ruby/ruby-2.2.3.tar.gz 
$ tar zxvf ruby-2.2.3.tar.gz 
$ cd ruby-2.2.3
$ ./configure --disable-install-rdoc
$ make && make install
```

### 于 mysql 中创建帐号及表
```sql
CREATE USER 'gitlab'@'localhost' IDENTIFIED BY 'gitlab';

CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';
```

### 添加 git 帐号并加入 sudoers
```shell
$ useradd --comment 'GitLab' git
$ echo "git ALL=(ALL)       NOPASSWD: ALL" >>/etc/sudoers
```

### 安装 gitlab
此处直接参考 [官网](https://about.gitlab.com/downloads/#centos6)

1. Install and configure the necessary dependencies
	```SHELL
	sudo yum install curl openssh-server openssh-clients postfix cronie
	sudo service postfix start
	sudo chkconfig postfix on
	sudo lokkit -s http -s ssh
	```
	
2. Add the GitLab package server and install the package （可事先修改为 [清华源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)）
	```
	curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
	sudo yum install gitlab-ce
	```	
	
3. Configure and start GitLab
	```
	sudo gitlab-ctl reconfigure
	```
		
### 配置原有 nginx
```conf
# gitlab socket location
upstream gitlab {
  # 7.x 
  # server unix:/var/opt/gitlab/gitlab-rails/tmp/sockets/gitlab.socket;
  # 8.0 
  server unix://var/opt/gitlab/gitlab-rails/sockets/gitlab.socket;
}

server {
  listen *:80;

  server_name gitlab.azlar.cc;   

  server_tokens off;     # don't show the version number, a security best practice
  root /opt/gitlab/embedded/service/gitlab-rails/public;

  # Increase this if you want to upload large attachments
  # Or if you want to accept large git objects over http
  client_max_body_size 250m;

  # individual nginx logs for this gitlab vhost
  access_log  /data/logs/nginx/gitlab_access.log;
  error_log   /data/logs/nginx/gitlab_error.log;

  location / {
    # serve static files from defined root folder;.
    # @gitlab is a named location for the upstream fallback, see below
    try_files $uri $uri/index.html $uri.html @gitlab;
  }

  # if a file, which is not found in the root folder is requested,
  # then the proxy pass the request to the upsteam (gitlab unicorn)
  location @gitlab {
    # If you use https make sure you disable gzip compression 
    # to be safe against BREACH attack

    proxy_read_timeout 300; # Some requests take more than 30 seconds.
    proxy_connect_timeout 300; # Some requests take more than 30 seconds.
    proxy_redirect     off;

    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Frame-Options   SAMEORIGIN;

    proxy_pass http://gitlab;
  }

  # Enable gzip compression as per rails guide: http://guides.rubyonrails.org/asset_pipeline.html#gzip-compression
  # WARNING: If you are using relative urls do remove the block below
  # See config/application.rb under "Relative url support" for the list of
  # other files that need to be changed for relative url support
  location ~ ^/(assets)/  {
    root /opt/gitlab/embedded/service/gitlab-rails/public;
    # gzip_static on; # to serve pre-gzipped version
    expires max;
    add_header Cache-Control public;
  }

  error_page 502 /502.html;
}
```

### 配置 gitlab.rb
```SHELL
$ vim /etc/gitlab/gitlab.rb

###### vim ######
nginx['enable'] = false
web_server['external_users'] = ['www', 'gitlab-www']
```
其中 `web_server['external_users']` 中的 `www` 为 `nginx` 进程的运行用户。

### gitlab-ctl reconfigure
编译后直接访问配置好的主机即可。


## 小记
本来在本机一台台式机上配置 ok 了，但是上到阿里云配完后发现内存不够，跑不动，一直报 502。

在阿里云上买机器配置目前不太现实，得想个办法，让本地局域网使用 gitlab，而让代码能够推送到线上生产环境。

另：
检测 `www`(nginx 所属用户组) 是否有权限访问 gitlab 主进程的 socket：
```
$ sudo -u www ls /var/opt/gitlab/gitlab-workhorse/socket
```

## 参考网站
1. [https://about.gitlab.com/downloads/#centos6](https://about.gitlab.com/downloads/#centos6)
2. [https://docs.gitlab.com/omnibus/settings/nginx.html](https://docs.gitlab.com/omnibus/settings/nginx.html)
3. [https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)
4. [http://blog.dianqk.org/2015/10/19/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1%20GitLab%20%E7%AD%89%E6%90%AD%E5%BB%BA%E8%BF%87%E7%A8%8B/](http://blog.dianqk.org/2015/10/19/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1%20GitLab%20%E7%AD%89%E6%90%AD%E5%BB%BA%E8%BF%87%E7%A8%8B/)
5. [https://www.liaohuqiu.net/cn/posts/non-bundled-web-server-for-gitlab/](https://www.liaohuqiu.net/cn/posts/non-bundled-web-server-for-gitlab/)