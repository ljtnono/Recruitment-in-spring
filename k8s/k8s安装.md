## Centos7 安装k8s教程

 ### 安装准备

| 操作系统 | ip地址         | 主机名 | 节点   |
| -------- | -------------- | ------ | ------ |
| centos7  | 192.168.137.57 |        | master |
| centos7  | 192.168.137.58 |        | node1  |
| centos7  | 192.168.137.59 |        | node2  |

### master和node基础配置

1、安装各种基础工具和配置yum源

```bash
# 这一步master和node节点都需要配置
yum install -y wget curl vim
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all && yum makecache && yum -y update
```

2、修改主机名以便区分

```bash
## master
echo "k8s-master" > /etc/hostname
## node1
echo "k8s-node1" > /etc/hostname
## node2
echo "k8s-node2" > /etc/hostname
```

3、修改hosts文件

```bash
# 这一步master和node节点都需要配置
vim /etc/hosts
# 新增以下内容
192.168.137.57  master
192.168.137.58  node1
192.168.137.59  node2
```

4、禁用一些基础设施

```bash
# 禁用防火墙
systemctl disable firewalld
systemctl stop firewalld
# 关闭swap
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
# 禁止selinux
# 临时禁用selinux
setenforce 0
# 永久关闭 修改/etc/sysconfig/selinux文件设置
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

5、安装并配置docker

```bash
# 卸载老版本
yum remove -y docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine

# 配置源
yum install -y yum-utils                
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo

# 安装
yum install -y docker-ce docker-ce-cli containerd.io

# 配置docker阿里源,可配可不配
cat>/etc/docker/daemon.json<<EOF
{
  "registry-mirrors": ["xxx"]
}
EOF

# 启动docker
systemctl start docker
```

6、安装k8s

需要安装的软件包：

* kubeadm：用来初始化集群的指令。
* kubelet：在集群中的每个节点上用来启动Pod和容器等。
* kubectl：用来与集群通信的命令行工具。

kubeadm **不能** 帮你安装或者管理 `kubelet` 或 `kubectl`，所以你需要 确保它们与通过 kubeadm 安装的控制平面的版本相匹配。 如果不这样做，则存在发生版本偏差的风险，可能会导致一些预料之外的错误和问题。 然而，控制平面与 kubelet 间的相差一个次要版本不一致是支持的，但 kubelet 的版本不可以超过 API 服务器的版本。 例如，1.7.0 版本的 kubelet 可以完全兼容 1.8.0 版本的 API 服务器，反之则不可以。

```bash
#配置kubernetes.repo的源，由于官方源国内无法访问，这里使用阿里云yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

#在所有节点上 kubelet、kubeadm 和 kubectl、kubernetes-cni
yum install -y kubelet kubeadm kubectl kubernetes-cni

#启动kubelet服务
systemctl enable kubelet && systemctl start kubelet

# 检查是否安装成功
kubelet --version
kubeadm version -o yaml
kubectl version --short
```





### 配置master节点

1、创建工作目录

```bash
mkdir /root/working
cd /root/working
```

2、创建kubeadm.conf配置文件



