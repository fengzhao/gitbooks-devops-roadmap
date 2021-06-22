# Python 环境的搭建

# 一、安装

## 1、源码编译安装

### CentOS

```bash
version=3.7.7
yum install -y sqlite-devel readline-devel tk-devel
wget https://www.python.org/ftp/python/$version/Python-$version.tgz
tar -xzf Python-$version.tgz
cd Python-$version
./configure --enable-optimizations
make
make install
```

### Ubuntu

```bash
version=3.7.7
apt update
apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev \
			libsqlite3-dev libreadline-dev libffi-dev wget libbz2-dev
wget https://www.python.org/ftp/python/$version/Python-$version.tgz
tar -xzf Python-$version.tgz
cd Python-$version
./configure --enable-optimizations
make
make install
```

# 二、Python的包管理器pip

## 1、 安装

### YUM

```
yum install -y epel-release ;\
yum install python-pip
```

#### APT

```
apt-get install python3-pip
```

## 2、更新

```bash
pip3 install -U pip
```

```bash
pthon3 -m pip install -U pip
```

