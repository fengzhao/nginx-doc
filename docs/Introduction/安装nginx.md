# 安装 nginx

nginx 可以以不同的方式安装，安装方式取决于当前的操作系统。

在 Linux 上，当然可以使用 yum、apt-get 等软件包管理工具来下载 Nginx，但是 Nginx 的很多模块并不是默认开启的，第三方模块很多也并不包含。

所以，如果想要开启内置的模块或编译第三方模块，还是需要编译 Nginx。



## 在 Linux 上安装

对于 Linux，可以使用 nginx.org 的 nginx [软件包](../其他/linux包.md)。

## 在 FreeBSD 上安装

在 FreeBSD 上，可以从 [软件包](http://www.freebsd.org/doc/handbook/pkgng-intro.html) 或通过 [ports](http://www.freebsd.org/doc/handbook/ports-using.html) 系统安装 nginx。ports 系统提供更大的灵活性，提供了各种选项可供选择。port 将使用指定的选项编译 nginx 并进行安装。

## 包管理器安装

```shell
# https://www.nginx.com/resources/wiki/start/topics/tutorials/install/

# 配置yum源：/etc/yum.repos.d/nginx.repo 

# CentOS
[nginx]
name=nginx repo
# 系统:CentOS7 架构:x86_64
baseurl=https://nginx.org/packages/centos/7/x86_64/

# 系统:CentOS8 架构:x86_64
# baseurl=https://nginx.org/packages/centos/7/x86_64/

# 系统:RHEL7 架构:x86_64
# baseurl=https://nginx.org/packages/rhel/7/x86_64/

# 系统:RHEL8 架构:x86_64
# baseurl=https://nginx.org/packages/rhel/7/x86_64/
gpgcheck=0
enabled=1

```


## 从源码安装

如果需要某些特殊功能软件包和 ports 无法提供，那么可以从源码中编译 nginx。虽然此方式更加灵活，但对于初学者来说可能会很复杂。更多信息请参阅 [从源码构建 nginx](../How-To/从源码构建nginx.md)。

```shell
# 添加运行nginx的用户
groupadd nginx
useradd -r -g nginx -s /bin/false nginx


# 安装依赖环境，确保编译正常
sudo yum install gcc gcc-c++ make automake autoconf libtool pcre* zlib openssl openssl-devel  pcre pcre-devel

sudo apt install zlib1g zlib1g-dev

# 在 http://nginx.org/en/download.html 里面下载 Nginx 源代码，永远下载最新的Stable release版本
wget http://nginx.org/download/nginx-1.21.6.tar.gz


# https://www.cnblogs.com/chrdai/p/11306728.html
nginx-1.21.4
├── CHANGES 		# 每个版本提供的特性和 bugfix，changelog文件
├── CHANGES.ru 		# 俄罗斯版本的 CHANGES 文件
├── LICENSE			# 开源许可文件
├── Makefile        # MAKE文件   
├── README          # README说明文件
├── auto            # 自动检测系统环境以及编译相关的脚本，辅助 configure 脚本执行的时候去判定nginx支持哪些模块，当前操作系统有什么样的特性可以供给nginx使用
├── conf            # 默认配置文件，方便运维配置，会把conf示例配置文件拷贝到安装目录
├── configure       # 命令脚本，用来生成中间文件，执行编译前的一个必备动作
├── contrib         # 提供了两个 pl 脚本和 vim 工具
├── html            # 一个 500 错误的默认页面，另一个是默认的 index 页面
├── man             # nginx 对 Linux 的帮助文件，man ./nginx.8
└── src             # nginx 核心源代码


# 配置 Vim 高亮, 如果 Vim 没有开启语法高亮的话，最好开启一下
cp -r contrib/vim/* ~/.vim
echo 'syntax on' > ~/.vimrc 


# configure配置，configure之后，会生成Makefile文件，用于编译和构建nginx
./configure   --user=nginx --group=nginx  --prefix=/usr/local/nginx/  --with-http_stub_status_module  --with-http_ssl_module

./configure --help # --help 命令可以查看配置脚本支持哪些参数选项

# 第一类配置参数

--prefix=PATH                      set installation prefix      # 一般指定这个路径就可以了，其他文件会在 prefix 目录下建立相应的文件夹
--sbin-path=PATH                   set nginx binary pathname
--modules-path=PATH                set modules path
--conf-path=PATH                   set nginx.conf pathname
--error-log-path=PATH              set error log pathname
--pid-path=PATH                    set nginx.pid pathname
--lock-path=PATH                   set nginx.lock pathname
--user=USER                        set non-privileged user for worker processes
--group=GROUP                      set non-privileged group for worker processes
--build=NAME                       set build name
--builddir=DIR                     set build directory



# 第二类配置参数
# 可以配置使用或不使用哪些模块，前缀通常是 with 和 with out。
## with开头的表示该模块默认是未开启的，可以使用--with开启。
## without开头的表示该模块默认是启用的，可以使用--without禁用。
## 第三方模块使用--add-module=PATH添加。如果支持动态加载，使用--add-dynamic-module=PATH添加。

--with-http_ssl_module             enable ngx_http_ssl_module
--with-http_v2_module              enable ngx_http_v2_module
--with-http_realip_module          enable ngx_http_realip_module
...
--without-http_charset_module      disable ngx_http_charset_module
--without-http_gzip_module         disable ngx_http_gzip_module
--without-http_ssi_module          disable ngx_http_ssi_module
...

# 编译并安装
make && make install 



# nginx服务配置文件，使用systemd
# /lib/systemd/system/nginx.service
# https://www.nginx.com/resources/wiki/start/topics/examples/systemd/

[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target
[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
[Install]
WantedBy=multi-user.target



# ubuntu

sudo apt-get install gcc make libpcre3 libpcre3-dev   zlib1g-dev openssl libssl-dev 


```

## Docker运行

利用 Docker 直接运行一个容器，然后将配置映射出来也是一种不错的选择：


```shell
$ docker run -d \
    --name nginx \
    -p 80:80 \
    -p 443:443 \
    -v $PWD/conf.d:/etc/nginx/conf.d \
    nginx

$ curl -v localhost
<html>
......
</html>

```

## 动态模块

动态模块，默认情况 nginx 是由许多核心模块构建的，那需要实现额外的功能就需要重新构建。但版本 1.9.11 之后，nginx 支持了`动态模块`，以下是 nginx 动态构建的模块：

nginx-module-geoip
nginx-module-image-filter
nginx-module-njs
nginx-module-perl
nginx-module-xslt
它可以通过单独的包进行发布，在使用时只需 nginx 加载即可：

# 作用域 main
load_module modules/ngx_mail_module.so;

## 原文档

[http://nginx.org/en/docs/install.html](http://nginx.org/en/docs/install.html)
