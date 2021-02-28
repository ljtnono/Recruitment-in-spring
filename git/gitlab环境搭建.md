## 拉取docker镜像

gitlab分为gitlab-ee和gitlab-ce两种，一般个人使用gitlab-ce就可以了，gitlab-ee需要付费

```shell
docker pull gitlab/gitlab-ce
```

## 设置gitlab存储数据文件夹

为了防止gitlab中的数据丢失，需要将容器中的部分文件夹映射到宿主机

```shell
export GIT_HOME=/home/software/gitlab
export PATH=$PATH:$GIT_HOME
```

**说明：**

* **$GIT_HOME/data    gitlab用于存放数据文件夹**
* **$GIT_HOME/config    gitlab用于存放配置文件文件夹**
* **$GIT_HOME/logs    gitlab用于存放日志文件文件夹**

## 运行启动

```shell
docker run --detach \
  --hostname gitlab.example.com \
  --publish 8443:443 --publish 8880:80 --publish 8222:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  --privileged=true \
  gitlab/gitlab-ce:latest
```

**说明：**

* **--hostname gitlab.example.com: 设置主机名或域名**
* **--publish 8443:443：将http：443映射到外部端口8443**
* **--publish 8880:80：将web：80映射到外部端口8880**
* **--publish 8222:22：将ssh：22映射到外部端口8222**
* **--name gitlab: 运行容器名**
* **--restart always: 自动重启**
* **--volume /srv/gitlab/config:/etc/gitlab: 挂载目录**
* **--volume /srv/gitlab/logs:/var/log/gitlab: 挂载目录**
* **--volume /srv/gitlab/data:/var/opt/gitlab: 挂载目录**
* **--privileged=true 使得容器内的root拥有真正的root权限。否则，container内的root只是外部的一个普通用户权限**

## 访问

gitlab启动成功后，访问浏览器http://ip:80，即可访问。为了使用域名访问，需要配置nginx

```shell
upstream gitlab{
    server 127.0.0.1:8880;
}

server {
    listen 80;
    server_name  gitlab.example.com;
    location / {
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_pass http://gitlab;
    }
}
```

nginx重启配置生效后，浏览器访问[http://gitlab.example.com](http://gitlab.example.com/) 即可正常访问。

首次访问需要为root用户设置密码，设置完成后需要登录，默认用户名为：root， 密码为刚刚设置的密码。

## 配置邮件服务器

想要让 GitLab 给你发送邮件，还要配置一下邮件服务器，这里以QQ邮箱的 IMAP/SMTP服务 来配置。

打开邮箱->设置->账户，然后开启 IMAP/SMTP服务，然后根据文档获取 授权码 ，这步比较重要。

然后跳转至挂载目录 `/srv/gitlab/config/` 编辑`gitlab.rb` 文件，找到 Email Settings的注释位置，然后修改以下内容：

```shell
### Email Settings
gitlab_rails['smtp_enable'] = true # 开启 SMTP 功能
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465 # 端口不可以选择587，测试过会发送邮件失败
gitlab_rails['smtp_user_name'] = "test@qq.com" # 你的邮箱账号
gitlab_rails['smtp_password'] = "1324dasd" # 授权码，不是密码
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = 'test@qq.com' # 发件人信息，必须跟‘smtp_user_name’保持一致，否则报错
gitlab_rails['smtp_domain'] = "qq.com" # 修改并不影响 
```

配置完成后保存，然后输入下面的命令使配置生效。

```shell
sudo docker exec gitlab gitlab-ctl reconfigure
```

使配置生效之后我们可以使用 gitlab 自带的工具进行一下测试。依次执行下面的命令：

```shell
# 开启 gitlab 的 bash 工具
$ docker exec -it gitlab bash

# 开启 gitlab-rails 工具
$ gitlab-rails console production

# 发送邮件进行测试
Notify.test_email('test_001@123.com', 'Message Subject', 'Message Body').deliver_now
```

测试完成之后退出gitlab的bash工具，重启 gitlab 即可。

```shell
docker restart gitlab
```

## 配置Git仓库访问路径

在之前第一次运行 gitlab 容器的时候，有一个参数 hostname 为 gitlab.example.com , 如果配置了域名可以忽略这一步，如果你没有配置相应域名的话，你的仓库的地址将会变为下面这样：

```shell
ssh : git@gitlab.example.com:test/test.git
http：gitlab.example.com/test/test.git
```

如果域名不存在的话，这个地址是无法进行 clone 的。

为了解决这个问题，我们可以设置成 IP 或 你配置了的域名来访问。

打开文件 `/srv/gitlab/config/gitlab.rb` 文件并找到

```shell
# external_url 'GENERATED_EXTERNAL_URL'
```

这行，去掉注释，并按照下面的格式修改。

```shell
# ip 形式
external_url 'http://192.168.1.44'

# 域名形式
external_url 'http://JemGeek.com'

# 子域名
external_url 'http://gitlab.JemGeek.com'

# 其他形式
external_url 'http://JemGeek.com/gitlab'

```

以上形式都是可以的。修改完成后，输入命令:

```shell
docker exec gitlab gitlab-ctl reconfigure
```

使配置生效，然后重启 gitlab 即可。

## gitlab-ctl 命令

```shell
docker exec gitlab gitlab-ctl stop # 停止
docker exec gitlab gitlab-ctl reconfigure # 重新配置
docker exec gitlab gitlab-ctl start # 启动
```

