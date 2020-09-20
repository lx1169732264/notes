# 安装



* yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
* yum -y install make zlib-devel gcc-c++ libtool openssl openssl-devel

前置依赖环境

* yum install git

* docker pull season/fastdfs

* mkdir -p /data/fdfs

  创建存储目录

* 安装libfatscommon

  git clone https://github.com/happyfish100/libfastcommon.git --depth 1

  cd libfastcommon/

  ./make.sh && ./make.sh install

  cd ..

* 安装fastdfs

  git clone https://github.com/happyfish100/fastdfs.git --depth 1

  cd fastdfs/

  ./make.sh && ./make.sh install #编译安装

  cd ..

* 安装nginx

  wget http://nginx.org/download/nginx-1.15.4.tar.gz #下载nginx压缩包

  tar -zxvf nginx-1.15.4.tar.gz #解压

  cd nginx-1.15.4/

  #添加fastdfs-nginx-module模块
  ./configure --add-module=/data/fdfs/fastdfs-nginx-module/src/ 

  make && make install #编译安装		出现错误则 ./configure 

  cd /usr/local 查看nginx是否安装成功，成功则显示nginx目录