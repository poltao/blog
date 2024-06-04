+++
title = "基于腾讯云cvm的云原生环境搭建"
description = "基于腾讯云cvm的云原生环境搭建，所有操作基于腾讯云 cvm 实例。"
date = 2021-12-14
[taxonomies]
tags = ["Linux", "VIM", "Commitizen", "NodeJS"]
+++

具体配置如下，操作系统为 TencentOS Server 3.1：:smile:

```bash
[root@VM-0-16-centos cms]$ cat /etc/motd
Welcome to TencentOS 3 64bit
Version 3.1 20210604
tlinux3.1-64bit-5.4.119-19.0006-20210623
```

## 1.安装 vim

主要是更新 vim 的版本，使其>8.0,这样某些插件才可以正常安装。

```bash
yum remove vim vi
rm -fr /usr/local/vim /usr/bin/vim
cd /tmp
wget https://github.com/vim/vim/archive/v8.2.0000.tar.gz
tar xzf  v8.2.0000.tar.gz
cd vim-8.2.0000/
./configure --prefix=/usr/local/vim  --with-features=huge --enable-multibyte --enable-gtk3-check  --enable-rubyinterp=yes --with-python3-command=python3 --enable-python3interp=yes --enable-perlinterp=yes --enable-luainterp=yes --enable-cscope
make && make install
ln -s /usr/local/vim/bin/vim  /usr/bin/vim
vim --version #验证是否安装成功
```

接下来快速安装基本插件：

```bash
cd ~
git clone https://gitcode.net/qq_41345173/tvim.git
cd tvim
cp .vimrc ~/.vimrc
mkdir -p ~/.vim/
cp -r colors/ ~/.vim/
cp -r autoload/ ~/.vim/
cp -r plugged/ ~/.vim/
cd ~/.vim/plugged
unzip plugged.zip
cd ~
rm -fr tvim
```

相关文章：
<https://blog.csdn.net/qq_41345173/article/details/120381818>

## 2.tlinux3 安装 docker

Tlinux3 和其他原生操作系统不同，有团队维护 tlinux 源，安装 docker-ce 的方式如下：

```bash
yum install tencentos-release-docker-ce
yum update
yum install docker-ce docker-ce-cli containerd.io
# 测试
docker version
```

配置 github 相关域名的 ip 地址，以进行加速访问 github 资源：

```bash
vim /etc/hosts
# 尾部添加如下内容
# 在网站https://www.ipaddress.com/获取最新ip
140.82.113.3 github.com
185.199.108.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.110.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```

Docker-compose 安装：

```bash
curl -LO "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

## 3.安装 golang

安装 golang

```bash
wget https://dl.google.com/go/go1.16.linux-amd64.tar.gz
sha256sum go1.16.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.16.linux-amd64.tar.gz
# 设置环境变量，在~/.bashrc中添加如下配置
export PATH=$PATH:/usr/local/go/bin
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct
export GOSUMDB=off
export GOPATH=/root/go
export PATH=$PATH:$GOPATH/bin
```

安装 proto 依赖，便于进行 RPC 开发

```bash
git clone https://github.com/protocolbuffers/protobuf
# v3.6.0+以上版本支持map解析，syntax=2、3消息序列化后是二进制兼容的，用root执行以下命令
cd protobuf
./autogen.sh
./configure
make
make install
ldconfig
protoc --version # 检测是否安装成功
go get github.com/golang/protobuf # 安装依赖包
# 安装gprc protocol插件
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
# 安装grpc-gateway插件
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest
```

## 4.安装 nginx

通过以下命令安装和启动 nginx:

```bash
yum install nginx
systemctl start nginx
# nginx默认监听端口为80，安装后可以通过/etc/nginx/nginx.conf配置文件修改监听的端口
# systemctl reload nginx可以一键重启nginx服务
```

## 5.安装 nodejs

首先在[官网下载页面](https://nodejs.org/zh-cn/download/)获取下载链接，然后下载安装：

```bash
mkdir -p /usr/local/nodejs
cd /tmp
curl -fLO https://nodejs.org/dist/v16.16.0/node-v16.16.0-linux-x64.tar.xz
tar -xJvf node-v16.16.0-linux-x64.tar.xz -C /usr/local/nodejs
echo 'export PATH=/usr/local/nodejs/node-v16.16.0-linux-x64.tar.xz/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

## 6.安装 commitizen 工具

[commitizen 工具](https://github.com/commitizen/cz-cli)是规范化 git 提交信息的 node 插件，安装流程如下：

```bash
npm install -g commitizen
echo '{ "path": "cz-conventional-changelog" }' >> ~/.czrc
```

**Pro tip:** You can use all the git commit options with <font color='red'>git cz</font>. For example: <font color='red'>git cz
--signoff.</font>