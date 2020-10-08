## 使用树莓派搭建NAS

### 准备工作

* 一块移动硬盘
* 一个树莓派
* 一根网线
* TF扩展卡和读卡器

### 安装镜像

#### 下载系统镜像

可以在[https://www.raspberrypi.org/downloads/](https://www.raspberrypi.org/downloads/)下载各种镜像，这里我使用的是ubuntu的镜像，因为ubuntu的操作相对来说比较简单，更容易上手。

#### 下载烧录软件



推荐使用官方推荐的Raspberry Pi Imager软件来进行镜像烧录，下载地址：[https://www.raspberrypi.org/downloads/](https://www.raspberrypi.org/downloads/)，当然也可以使用其他的烧录软件进行烧录。

### 烧录

1. 解压镜像文件

   ![解压镜像文件](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422210807927.png)

2. 使用烧录工具Raspberry Pi Imager进行烧录

   ![选择镜像](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422211040120.png)

   ![使用自定义镜像](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422211225935.png)

![选择刚才解压的镜像文件](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422211257037.png)

![选择sd卡](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422211632301.png)

![选中SD卡](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422211841292.png)

![写入sd卡](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422211921322.png)

3. 开启ssh

   进入到sd卡的boot里面，新增一个ssh文件（注意：文件名就是ssh，没有后缀）用来开启ssh远程连接功能。

### 启动树莓派

烧录完成之后，需要将sd卡插入到树莓派中，然后将树莓派插上电源，接上网线，将移动硬盘接入树莓派。

### 查看树莓派的IP并使用ssh连接

首先需要从路由器中获取到树莓派的ip地址，浏览器输入192.168.1.1可以进入路由器管理界面

![获取树莓派ip地址](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422213254001.png)

然后使用finalshell进行连接，初始账号密码都是ubuntu，第一次连接会让你修改密码

![修改密码](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422213421333.png)

改完密码之后就可以重新连接。

### 修改apt软件源

由于树莓派的apt软件源默认是国外的地址，导致我们下载软件的速度很慢，所以我们可以修改apt的软件源为阿里的镜像源，这样就可以加快下载速度

```bash
# 备份
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 修改源
sudo vim /etc/apt/sources.list
# 将文件中所有的内容注释掉，然后在最下面新增下面的内容
deb http://mirrors.aliyun.com/ubuntu-ports eoan main restricted
deb http://mirrors.aliyun.com/ubuntu-ports eoan-updates main restricted
deb http://mirrors.aliyun.com/ubuntu-ports eoan universe
deb http://mirrors.aliyun.com/ubuntu-ports eoan-updates universe
deb http://mirrors.aliyun.com/ubuntu-ports eoan multiverse
deb http://mirrors.aliyun.com/ubuntu-ports eoan-updates multiverse
deb http://mirrors.aliyun.com/ubuntu-ports eoan-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports eoan-security main restricted
deb http://mirrors.aliyun.com/ubuntu-ports eoan-security universe
deb http://mirrors.aliyun.com/ubuntu-ports eoan-security multiverse
# 更新源
sudo apt-get update
# 更新源的时候会出来一个窗口，这里选择yes
```

### 挂载移动硬盘

在进行samba安装和配置之前，我们需要将移动硬盘挂载到树莓派上，并且设置为开机自动挂载

```bash
# 首先查看一下移动硬盘的挂载路径
sudo fdisk -l
# 下面是我的硬盘的状况
Disk /dev/mmcblk0: 14.5 GiB, 15552479232 bytes, 30375936 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xda84cd12

Device         Boot  Start      End  Sectors  Size Id Type
/dev/mmcblk0p1 *      2048   526335   524288  256M  c W95 FAT32 (LBA)
/dev/mmcblk0p2      526336 30375902 29849567 14.2G 83 Linux


Disk /dev/sda: 931.5 GiB, 1000204883968 bytes, 1953525164 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc1efe78c

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1  *     2048 1953521663 1953519616 931.5G  7 HPFS/NTFS/exFAT

# 上面发现/dev/sda1 是我们的移动硬盘

# 创建一个目录用于挂载
sudo mkdir /MyData
# 修改目录的所有者和权限
sudo chown -R ubuntu:ubuntu /MyData
sudo chmod 755 -R /MyData
# 挂载目录
sudo mount -t auto /dev/sda1 /MyData
```

挂载完毕之后，我们发现重启树莓派之后，每次都要执行挂载命令，所以我们需要让树莓派启动的时候自动挂载

```bash
# 首先查看移动盘的挂载信息
blkid /dev/sda1
## 信息如下
## /dev/sda1: LABEL="MyDataPan" UUID="84C4EED6C4EECA0E" TYPE="ntfs" PARTUUID="c1efe78c-01"
sudo vim /etc/fstab
# 新增一行
LABEL=MyDataPan	/MyData	ntfs	defaults	0	0
# 这里添加完毕之后，一定要用mount -a 命令测试一下，否则如果不成功，将导致树莓派无法开机成功（血的教训）
sudo mount -a
# 如果没有错误提示，那么应该就没问题了
```

### 安装samba

samba是一种共享协议，可以将局域网中的硬盘共享到局域网中的主机中，也就是我们经常在windows中使用的共享文件夹，现在我们需要先安装这个包。

```bash
sudo apt install samba
sudo apt install samba-common-bin
```

### 配置samba

安装完samba之后，我们需要对samba进行一些配置，方便后面能够在windows中使用

```bash
# 备份
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
# 修改配置
sudo vim /etc/samba/smb.conf
# 找到 Share Definition 下面的read only选项，修改为read only = no
read only = no
# 在最后一行添加配置
[share]
   comment = this is MyDataPan
   path = /MyData
   valid users = @users
   force group = users
   create mask = 0660
   directory mask = 0771
   read only = no
# 重启samba
sudo samba restart
```

### 新增samba用户

配置完samba之后，我们需要配置一个新的用户用于其他内网设备访问

```bash
# 新增用户及用户组，这里users是前面samba设置的users用户组
sudo useradd share -m -G users
# 修改密码
sudo passwd share
# 将用户添加到samba 用户中去，并设置访问密码
sudo smbpasswd -a share
# 输入密码之后就OK了
```

至此，所有的配置完毕，可以打开windows进行测试了

![成功访问share文件夹](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422230349231.png)





### 后记

使用share文件夹的时候，发现如果手贱点了其他用户登录的话导致出现不能登录

![登录share异常](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422230541075.png)

解决方案：

![登录share异常解决方案](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200422230714115.png)