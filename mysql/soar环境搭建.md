

SOAR支持Linux、Mac、Window环境，可以在Linux、Mac、Window上安装使用。下面以Linux版本为例介绍安装使用过程。

## 安装GO环境

SOAR的使用依赖Go语言，所以在安装使用之前，先安装Go

```bash
# 下载go语言
wget https://dl.google.com/go/go1.16.4.linux-amd64.tar.gz
# 配置环境变量,将下面<go>改为go安装目录再执行
echo "# 配置go环境" >> /etc/profile
echo "export GO_HOME=<go>" >> /etc/profile
echo 'export PATH=$PATH:$GO_HOME/bin' >> /etc/profile
source /etc/profile
```

Go安装完成后，输入 go version ，如果出现版本信息，则表示安装成功

```bash
go version
# go version go1.16.4 linux/amd64
```

## 安装SOAR

```bash
# 下载soar,最好将soar放在特定目录下
wget https://github.com/XiaoMi/soar/releases/download/0.11.0/soar.linux-amd64 -O soar
```

测试SOAR是否安装成功

```bash
./soar --version
# Version: 2019-01-21 16:54:09 +0800 0.11.0-16-gc12ae96
# Branch: master
# Compile: 2019-01-21 16:55:46 +0800 by go version go1.10.7 linux/amd64
# GitDirty: 0
```

## 安装SOAR的图形化界面程序

soar-web程序依赖python运行，所以安装之前需要先安装python3环境

```bash
# 安装python
yum install -y python36 python36-pip
# 一般情况下python3 
```

安装soar-web

```bash
wget https://codeload.github.com/xiyangxixian/soar-web/zip/master -O soar-web-master.zip
```



